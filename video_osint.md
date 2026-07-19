# Video OSINT

## Challenge Description
Identify the location and date of the event depicted in the video matching a given ciphertext string:
`BQS?8F#ks-GB\6`H#IhIF^eo7@rH3;G@>T'BKpZ':01;:GVh!HAQ!.fF?MN>Er`

## Analysis & Solution
1. **Decode the ciphertext**:
   Using Ascii85 (Base85) decoding on the provided string yields:
   `https://www.youtube.com/watch?v=NWQwx4-MeRg&t=65s`
2. **Examine the Video**:
   The video is titled *"Afghanistan, 2009: a Year both Deadly and Decisive"*. Around the 65-second mark, U.S. soldiers are shown playing basketball in front of a military outpost with mountains in the background.
3. **OSINT Investigation**:
   A search for this iconic photograph shows it was taken at **Camp Bostick** (in the Naray/Nari district of Kunar Province, Afghanistan) on **April 16, 2009**.

## Flags (but not working)
- `athena{Camp Bostick_2009-04-16}`
- `athena{Naray_2009-04-16}`
