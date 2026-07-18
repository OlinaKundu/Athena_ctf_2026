# Global Blue Police Salute (OSINT)

## Challenge Description
Identify the creator (artist or account) behind a viral image from the "Global Blue Police Salute" meme (provided as [Policia.png](file:///e:/ctf/Policia.png)).

## Analysis & Solution
1. **Context Identification**:
   - The image depicts two police cars parked in front of a building illuminated in blue light at night, with a Slovenian flag and a European Union flag visible in the background.
   - A web search for the phrase `"Global Blue Police Salute"` reveals it is an international INTERPOL initiative held on the International Day of Remembrance for Fallen Officers (March 7) where member countries illuminate landmarks in blue.
   - The police cars have markings showing `...A.SI` and `POLICIJA`. Combining this with the flags, we locate the setting as the **Škofja Loka Police Station** (*Policijska postaja Škofja Loka*) in Slovenia.

2. **Source Analysis**:
   - We find the original high-resolution photo on the INTERPOL website under their event gallery:
     `https://www.interpol.int/var/interpol/storage/images/3/0/4/0/420403-1-eng-GB/478a527c6dfc-SLOVENIA-Skofja-Loka-Policijska-postaja-Skofja-Loka-Slovenska-Policija-1.JPG`

3. **Metadata Extraction**:
   - By downloading the original JPEG and inspecting its EXIF metadata using Python's `PIL` library, we extract key photographer information:
     - **Artist (Tag 315)**: `zdomen`
     - **Copyright (Tag 33432)**: `fotodins`
     - **Camera**: `SONY ILCE-6700`
     - **Software**: `Adobe Lightroom 10.4.2 (iOS)`

4. **Creator Identification**:
   - `fotodins` refers to the Slovenian photography duo **Foto DINS** (comprising Domen and Stella). 
   - The artist behind the camera who uploaded/captured the original post is **Domen**, represented by the username/artist tag `zdomen`.

## Flag
`athena{zdomen}`
