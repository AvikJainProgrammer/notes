https://www.youtube.com/watch?v=BT2Cv-Tjq7Q
LLVM standardises turning source code into machine code. 
Created in 2003 by Chris Latner 
Used in Clang, C, C++, Rust, Swfit and Julia. 
It helps represent High level source code in a code agnostic code called IR(Inter mediate representaion).
	- IR are independent of any machine architecture. 
Different languages like CUDA(Cuda is a language ?) and ruby produce the IR so they can share tools for analysis before they are turned to machine code. 
A compliner can be divided into 3 parts
	- The front end that converts the code into IR. 
	- The middle analyses and optamises this generated code. 
	- The backend converts the IR into native machine code. Say code is ARMx86 co the back will be specifically for that. 
To make you own programming language you need to
	- Install LLVM 
	- Create a C++ file
		- Envision the programming language
		- Create a LEXER that takes raw source code and converts it into a collection of tokens. 
			- like Identifier, keyword, separator, operator, literal etc
		- Define an AST(Abstract Syntax tree) to represent the structure of the code. 
			- This is accomplisehd by giving each node its class. 
		- Then create a parser to loop over each token and create the abstract syntax tree. 
		- Now we import bunch of LLVM primitives to generate a bunch of Intermediate representation code. 
		- Each type in AST is given a type called code gen. 
		- Each type is given a method called codegen which returns LLVM value object
			- LLVM value object returns a single assignment register
			- Single assignment register is a variable for the complier that can be only assigned once. 
	- Then an optamization tool is used to optamzie the generated code.
		- It does things like 
			- Dead code elemination
			- Scaler replacement of aggrigates
	- Then we come to backend where we write a module that converts the IR to emit object code that can run on any architecture. 


