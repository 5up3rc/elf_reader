How ELF strings are replaced
============================

 1. Identify all string table sections.

 2. Duplicated and split string tables into arrays of strings. Keep track of
    the original start and end indices of each string.

 3. For each string in each string array, see if it matches the search regex,
    and replace it if so.

 4. Rejoin the duplicated string arrays into new string tables. Remember: start
    with a null byte, end with a null byte, and insert a null byte between all
    strings. Keep track of the start and end indices of each of these new
    strings.

 5. Append the new string table sections to the end of the file.

 6. Change the offset and length of the original string table section headers
    to refer to the locations and sizes of the updated string tables (now at
    the end of the file). Don't forget to update the Virtual Address field
    in addition to the file offset.

 7. For each string reference in the file, look at the two offset maps created
    when parsing the original string tables and rebuilding the updated ones.
    Update the references to the correct new offsets.

 8. Add a new loadable read only data segment that will encompass the string
    tables at the end of the file. Make sure it uses the correct virtual
    address and file offsets.

 9. Add the new segment to the segment header table. Write this to the end of
    the file, too. Expand the size of the new read-only data segment containing
    the updated string tables to also include the size of these new segment
    headers. Make sure the new program header table starts at an 8-byte aligned
    address.

 10. In the new segment header table, update the program headers segment to
     encompass the relocated program headers.

 11. Write the result to the new output ELF file.

Known fields which refer to string table entries
================================================

 - The "Name" field in section headers

 - The "Name" field in symbol table entries

 - The "Name" field in ELF32Verdaux structures, in the `.gnu_version_d`
   sections. (The version symbol table contains 16-bit entries showing only
   local, global, or user-defined scope).

 - In `.gnu_version_r` sections:

     - The "VNFile" field in ELF32Verneed structures

     - The "Name" field in ELF32Vernaux structures

Fields which *may* refer to strings, pending further investigation
==================================================================

 - The "Value" field in symbol table entries. If it contains the virtual
   address or offset of a string table entry, should it be adjusted?