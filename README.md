# AssemblyToolchains
This serves as the main repo for ITSC 204 Assignment 1

# Contributor List (alphapbetical order, first name):
Lubos Kuzma (https://www.linkedin.com/in/lubos-kuzma-0719a586)  


# Suggested TODO:
## x86 Toolchain:
- make 64bit the default

- #! /bin/bash

# Created by Lubos Kuzma
# ISS Program, SADT, SAIT
# August 2022
Hassan Olowookere

BITS=True

if [ $# -lt 1 ]; then
    # ... (existing usage message)
    exit 1
fi

# ... (existing script lines)

while [[ $# -gt 0 ]]; do
    case $1 in
        # ... (existing cases)

        -64|--x86-64)
            BITS=True
            shift # past argument
            ;;

        # ... (existing cases)
    esac
done

# ... (existing script lines)

if [ "$BITS" == "True" ]; then
    nasm -f elf64 $1 -o $OUTPUT_FILE.o && echo ""
    ld -m elf_x86_64 $OUTPUT_FILE.o -o $OUTPUT_FILE && echo ""
    qemu-x86_64 $OUTPUT_FILE && echo ""
else
    nasm -f elf $1 -o $OUTPUT_FILE.o && echo ""
    ld -m elf_i386 $OUTPUT_FILE.o -o $OUTPUT_FILE && echo ""
    qemu-i386 $OUTPUT_FILE && echo ""
fi

# ... (remaining script lines)

