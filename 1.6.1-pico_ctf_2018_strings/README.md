# Pico Ctf 2018 Strings Challenge

  ## Objective

  We have to find the flag inside the file `strings` without executing it.

  This ctf requires to use the strings command on linux.
  After reading the manpages for strings : `man strings` we can try running this command: 

  `strings --all strings | grep -iE "ctf|flag|code|password|pico"`

  ### -all option

  The `-all` option is required because by default GNU strings is ran with the option `-d` (or `--data`) which searches for strings only inside loaded and **initialized segments**, for example:
  - `.data` segment -> contains static and initialized data, where static data is data that persist in memory even after a function termination and initialized data are all the non-zero initialized data. In c++ and c even if a variable is initialized to zero by the coder directly that data is still saved as it was an uninitiliazed data (saved inside .bss segment).
  - `.text` -> Contains the machine code that is executed by the CPU. `strings` command can convert raw bytes into ascii (or UTF) characters.
  - `.rodata` -> Contains constant, initialized data that the program CANNOT modify. rodata stands for read only data indeed. 

  **Other segments**:\
  - `.bss` -> The unitialized static variables (all set to zero) live inside the `.bss` segment.
  - `Heap` -> Dynamic allocated memory
  - `Stack` -> Functions local variables, parameters, function calls (i mean that for each function there is a stack frame), return addresses, saved CPU registers, addresses pointed to by stack base and top pointers (rsp, rbp).

  ### -i -> ignore case
  ### E -> Extended Regex (allows me to use | in the regex for example)

  # END

  Let's execute the command:

  ```
  ◎ strings --all strings | grep -iE "ctf|flag|code|password|pico"
  YzOejwCTF3GVzbdb8PkOKp1cKvAwEUvRSOLLm1yFFETiT
  picoCTF{sTrIngS_sAVeS_Time_3f712a28}
  CAFeDDajt1StD9aJRCFN12mQcTf0LDSGHDCkch6XJueHm4ZgGG
  7Oqu9T7p8SAoQcOcQVHM46k1xpt1M6Iu2ag4dw1OFCTFRbv6
  7AnHzkVvoAlt4LXrfcTFMxJZPPcGK0cpVirZLhlr3YPbM9
  dcvHl1RXYPwi3hHgY5XCtfMcD1jpW188VqnycdOJPOGp5eAfcNJZerrcSFb5P4asO
  tedjxd10rPQo8dZj9KructfPPqrKBISAAMrwGWo1vsSou
  WbTgaBs2l8JKaZvv9BCKCoWICtFXwL2gmkKnxUP
  3cTFM5KUolv3lITluIDU148Z6xSdagGvE6iQptUOHf7p
  a53dOCtFgCVqLcSEaPxZXj5QIlxKv4Ne41OFp7ZHyfEX
  nbJe71FAFsHSnGeRubCtfWhXhfqWc0z50A5UxAYf4awL7
  Mf9MFZCIv8gY7faUsaeAx8oEI8pvFLAGYZSFekIv
  0KLthcTFnFPvAM2UW7Ok3nnQRettkQpQ1nRh9t
  ```

  The flag: picoCTF{sTrIngS_sAVeS_Time_3f712a28}
  
  si gode, bye bye
