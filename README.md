#!/bin/bash

# Periodic Table Element Lookup Script
# Queries the periodic_table database for element information

# Check if an argument was provided
if [[ -z "$1" ]]; then
  echo "Please provide an element as an argument."
  exit
fi

# Set up the database command with proper connection parameters
PSQL="psql --username=freecodecamp --dbname=periodic_table -t --no-align -c"

# Check if the input is numeric (atomic number) or string (symbol/name)
if [[ "$1" =~ ^[0-9]+$ ]]; then
  # Input is a number, treat as atomic_number
  ELEMENT=$($PSQL "SELECT e.atomic_number, e.symbol, e.name, t.type, p.atomic_mass, p.melting_point_celsius, p.boiling_point_celsius
FROM elements e
JOIN properties p ON e.atomic_number = p.atomic_number
JOIN types t ON p.type_id = t.type_id
WHERE e.atomic_number = $1")
else
  # Input is a string, could be symbol or name
  ELEMENT=$($PSQL "SELECT e.atomic_number, e.symbol, e.name, t.type, p.atomic_mass, p.melting_point_celsius, p.boiling_point_celsius
FROM elements e
JOIN properties p ON e.atomic_number = p.atomic_number
JOIN types t ON p.type_id = t.type_id
WHERE e.symbol = '$1' OR e.name = '$1'")
fi

# Check if element was found
if [[ -z "$ELEMENT" ]]; then
  echo "I could not find that element in the database."
  exit
fi

# Parse the result
IFS='|' read -r ATOMIC_NUMBER SYMBOL NAME TYPE ATOMIC_MASS MELTING_POINT BOILING_POINT <<< "$ELEMENT"

# Format and output the element information
echo "The element with atomic number $ATOMIC_NUMBER is $NAME ($SYMBOL). It's a $TYPE, with a mass of $ATOMIC_MASS amu. $NAME has a melting point of $MELTING_POINT celsius and a boiling point of $BOILING_POINT celsius."

