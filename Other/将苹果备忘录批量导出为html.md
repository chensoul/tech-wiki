NotesExporter.workflow

```bash
set exportFolder to (choose folder) as string
-- Simple text replacing
on replaceText(find, replace, subject)
	set prevTIDs to text item delimiters of AppleScript
	set text item delimiters of AppleScript to find
	set subject to text items of subject
	
	set text item delimiters of AppleScript to replace
	set subject to "" & subject
	set text item delimiters of AppleScript to prevTIDs
	
	return subject
end replaceText
-- Get an HTML file to save the note in.  We have to escape
-- the colons or AppleScript gets upset.
on noteNameToFilePath(noteName, tag)
	global exportFolder
	set strLength to the length of noteName
	
	if strLength > 250 then
		set noteName to text 1 thru 250 of noteName
	end if
	
	tell application "Finder"
		set convertedFolderPath to exportFolder & tag & ":"
		if (exists (folder convertedFolderPath)) then
			set outputFolder to convertedFolderPath
		else
			make new folder at exportFolder with properties {name:tag}
			set outputFolder to the result as string
		end if
	end tell
	
	set fileName to (outputFolder & replaceText(":", "_", noteName) & ".html")
	return fileName
end noteNameToFilePath
tell application "Notes"
	
	repeat with theNote in notes of default account
		
		--repeat with theNote in notes in folder "New Folder" of default account
		set noteLocked to password protected of theNote as boolean
		set modDate to modification date of theNote as date
		set creDate to creation date of theNote as date
		
		set noteID to id of theNote as string
		set oldDelimiters to AppleScript's text item delimiters
		set AppleScript's text item delimiters to "/"
		set theArray to every text item of noteID
		set AppleScript's text item delimiters to oldDelimiters
		
		if length of theArray > 4 then
			-- the last part of the string should contain the ID
			-- e.g. x-coredata://39376962-AA58-4676-9F0E-6376C665FDB6/ICNote/p599
			set noteID to item 5 of theArray
		else
			set noteID to ""
		end if
		
		if not noteLocked then
			-- file name composed by id and note title to overcome overwriting files
			set fileName to (name of theNote as string) as string
			
			set theContainer to container of theNote
			
			-- export the folder containing the notes as tag in bear
			-- the try catch overcome a 10.15.7 bug with some folders
			try
				if theContainer is not missing value then
					set tag to name of theContainer
					set theText to ("" & theText & "
#" & tag & "#") as string
				end if
			end try
			
			set filepath to noteNameToFilePath(fileName, tag) of me
			set noteFile to open for access filepath with write permission
			set theText to body of theNote as string
			
			write theText to noteFile as Unicode text
			close access noteFile
			
			tell application "Finder"
				set modification date of file (filepath) to modDate
			end tell
		end if
		
	end repeat
	
end tell
```

解决乱码问题：

```bash
find . -name *.html -type f | \
(while read file; do
iconv -f UTF-16 -t UTF-8 "$file" > "${file}.bak";
mv "${file}.bak" "$file"
sed -i "" 's#<h1><br></h1>##g' $file
done);
```
