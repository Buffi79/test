#!/bin/bash
dest="testing" # destination directory
file="example.rar"

(
  fc="$(unrar l "$file" | wc -l)" # get file count inside RAR file
  fc="$((fc-10))" # file count has 10 extra lines, remove them
  i=0  # index counter

  # I have to redirect the output from 0 to 1 because when there's a
  # "file already exists" kind of error, unrar halts the execution of the
  # script to wait for user input, so I have to process that message inside
  # of the loop
  unrar x "$file" "$dest" 0>&1 2>&1 | while IFS="" read -r in ; do

    # we got a "file already exists" message?
    if [[ "$f" == *"already"* ]]; then
      echo "Error"  # display an error message
      break         # exit the while loop
    fi

    # extract the pathname of file being extracted
    f=$(echo $in | cut -d ' ' -f 2)

    echo "XXX"
    echo "$f..." # Display file name in the dialog box
    echo "XXX"
    echo "$pct %" # Show percent
    pct="$(( 100*(++i)/fc ))" # Calculate current percent
  done

  exit ${PIPESTATUS[0]} # exit with status message we got from unrar

) | dialog --title "Extracting Files" --gauge "Please wait" 8 75 0

# if PIPESTATUS[0] is not equal to zero, it means there was an error
if [[ "${PIPESTATUS[0]}" -ne 0 ]]; then
  echo "There was an error." # display a message
  exit 1  # exit with status 1
fi

# if command was successful, display a message  
if (( "$?" == 0 )); then
  echo "Command executed correctly $?"
fi
