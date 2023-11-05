#!/bin/bash


# Created by Lubos Kuzma
# Updated by Hassan Olowookere
# ISS Program, SADT, SAIT
# November 2023

# Check if the number of arguments is less than 1
if [ $# -lt 1 ]; then
	# Print usage information
	echo "Usage:"
	echo ""
	echo "x86_toolchain.sh [ options ] <assembly filename> [-o | --output <output filename>]"
	echo ""
	echo "-v | --verbose                Show some information about steps performed."
	echo "-g | --gdb                    Run gdb command on executable."
	echo "-b | --break <break point>    Add breakpoint after running gdb. Default is _start."
	echo "-r | --run                    Run program in gdb automatically. Same as run command inside gdb env."
	echo "-q | --qemu                   Run executable in QEMU emulator. This will execute the program."
	echo "-32| --x64              Compile for 32bit (x64) system."
	echo "-o | --output <filename>      Output filename."
	# Exit the script with an error code
	exit 1
fi

# Initialize variables
GDB=False
OUTPUT_FILE=""
VERBOSE=False
BITS=True
QEMU=False
BREAK="_start"
RUN=False

# Parse command line arguments
while [[ $# -gt 0 ]]; do
	case $1 in
		-g|--gdb)
			GDB=True
			shift # past argument
			;;
		-o|--output)
			OUTPUT_FILE="$2"
			shift # past argument
			shift # past value
			;;
		-v|--verbose)
			VERBOSE=True
			shift # past argument
			;;
		-32|--x64)
			BITS=false
			shift # past argument
			;;
		-q|--qemu)
			QEMU=True
			shift # past argument
			;;
		-r|--run)
			RUN=True
			shift # past argument
			;;
		-b|--break)
			BREAK="$2"
			shift # past argument
			shift # past value
			;;
		-*|--*)
			echo "Unknown option $1"
			exit 1
			;;
		*)
			POSITIONAL_ARGS="$1" # save positional arg
			shift # past argument
			;;
	esac
done

# Check if the input file exists
if [[ ! -f $POSITIONAL_ARGS ]]; then
	echo "Specified file does not exist"
	exit 1
fi

# Set the output filename based on the input filename if not provided
if [ "$OUTPUT_FILE" == "" ]; then
	OUTPUT_FILE=${POSITIONAL_ARGS%.*}
fi

# Print the script configuration if verbose mode is enabled
if [ "$VERBOSE" == "True" ]; then
	echo "Arguments being set:"
	echo "	GDB = ${GDB}"
	echo "	RUN = ${RUN}"
	echo "	BREAK = ${BREAK}"
	echo "	QEMU = ${QEMU}"
	echo "	Input File = $POSITIONAL_ARGS"
	echo "	Output File = $OUTPUT_FILE"
	echo "	Verbose = $VERBOSE"
	echo "	32 bit mode = $BITS" 
	echo ""

	echo "NASM started..."
fi

# Assemble the code based on the selected architecture
if [ "$BITS" == "True" ]; then
	nasm -f elf64 $POSITIONAL_ARGS -o $OUTPUT_FILE.o && echo ""
elif [ "$BITS" == "False" ]; then
	nasm -f elf $POSITIONAL_ARGS -o $OUTPUT_FILE.o && echo ""
fi

# Print messages if verbose mode is enabled
if [ "$VERBOSE" == "True" ]; then
	echo "NASM finished"
	echo "Linking ..."
fi

# Link the object file based on the selected architecture
if [ "$BITS" == "True" ]; then
	ld -m elf_x86_64 $OUTPUT_FILE.o -o $OUTPUT_FILE && echo ""
elif [ "$BITS" == "False" ]; then
	ld -m elf_i386 $OUTPUT_FILE.o -o $OUTPUT_FILE && echo ""
fi

# Print message if verbose mode is enabled
if [ "$VERBOSE" == "True" ]; then
	echo "Linking finished"
fi

# Start QEMU if specified
if [ "$QEMU" == "True" ]; then
	echo "Starting QEMU ..."
	echo ""
	if [ "$BITS" == "True" ]; then
		qemu-x86_64 $OUTPUT_FILE && echo ""
	elif [ "$BITS" == "False" ]; then
		qemu-i386 $OUTPUT_FILE && echo ""
	fi
	exit 0
fi

# Run GDB if specified
if [ "$GDB" == "True" ]; then
	gdb_params=()
	gdb_params+=(-ex "b ${BREAK}")
	if [ "$RUN" == "True" ]; then
		gdb_params+=(-ex "r")
	fi
	gdb "${gdb_params[@]}" $OUTPUT_FILE
fi
