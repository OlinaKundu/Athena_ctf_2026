# USBStorage Residue (Forensics)

## Challenge Description
A PCAP capture file `usbstorage_rotated.pcap.gz` capturing raw USB bulk traffic. The goal is to analyze the traffic and extract the hidden flag.

## Analysis & Solution
1. **Extraction**:
   Decompressing the PCAP file yields `usbstorage_rotated.pcap`.
2. **Protocol Identification**:
   Inspecting the packet data reveals raw USB Mass Storage Bulk-Only Transport (BOT) transactions:
   - Command Block Wrappers (CBW, signature `USBC`)
   - Command Status Wrappers (CSW, signature `USBS`)
   - Intermediate data blocks
3. **Transaction Parsing**:
   Analyzing the SCSI commands shows a sequence of `WRITE (10)` (opcode `0x2a`) operations to sector regions starting around Logical Block Address (LBA) `0x90bb50` (sectors 50..63).
   There are two distinct sets of writes to these sectors:
   - **Set A** (first set of writes): An initial file `flag.gz` was written.
   - **Set B** (second set of writes): The file `flag.gz` was overwritten with updated content.
4. **Reassembly**:
   Reassembling the sectors in physical LBA order for both sets of writes produces two tar archives containing `flag.gz`.
5. **Flag Extraction**:
   - Extracting `flag.gz` from **Set A** yields a dummy flag: `athena{n0t_th3_r34l_fl4g}`.
   - Extracting `flag.gz` from **Set B** yields the real recovery key / flag:
     `athena{62d635d2c2f5bf36c7b78859069fd818}`

## Flag
`athena{62d635d2c2f5bf36c7b78859069fd818}`
