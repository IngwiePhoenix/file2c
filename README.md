# file2c - convert file to C source

This is a very, very simple tool. To compile it, just compile it. :)

	g++ file2c.cpp -o f2c
	
Then just use it like:

	f2c foo.txt fooFile > foo.c
	
Now you can compile foo.c and access it like any regular char array. You could serialize a zip archive in there, or things like that. Just keep in mind, that this is without a header! So to use it properly,you may want to create a header and declare these as "external". Therefore, the variable name appears at the top of the file after a // comment.

An example header may look like this:

	#ifndef HAVE_FILES_H
	#define HAVE_FILES_H
	
	// You can generate this. Each file starts with: // varName
	// where varName is the original variable name.
	
	extern char[] myFile;
	
	#endif
	
Then you can include that header into your code, so it is aware of the variables.

For an example usecase, consider that we are in a situation where we want to compile a zip file into our binary - for some reason, whatever. Then you would use generator rules, as the following. I am using Ninja here, so look up ninja to do that

	rule f2c
  	  command = f2c $in $varName > $out && echo "$varName" >>varList.txt
      description = Creating C object: $out
	rule f2c_header
      command = echo "// file2c header" > $out; $
        && echo "#ifndef HAVE_FILES_H" >> $out; $
        && echo "#define HAVE_FILES_H" >> $out; $
        && echo "" >> $out;
        && for line in "$(cat varList.txt)"; do echo "extern char[] $line;" >> $out; done;
        && echo "" >> $out;
        && echo "#endif" >> $out;
	rule cxx
      command = g++ $in -o $out
      description = Compiling: $in
  
	build f2c: file2c.cpp
	build myFiles.zip.c: f2c myFiles.zip | f2c
	build files.h: f2c_header | f2c myFiles.zip.c
	
A small tool, to do nice things. Have fun. ^.^
