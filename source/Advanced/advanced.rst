Download
++++++++++++++++

* I-DLV is a full-fledged Answer Set Programming and Datalog reasoner.
  It supports the ASP-Core-2 standard language; nevertheless, the system supports additional linguistic extensions such as list terms and external atoms. 
* I-DLV is also a full-fledged deductive database system, supporting query answering powered by the Magic Sets technique. 
  I-DLV has also been integrated as the grounding module of the completely renewed second version of the logic-based Artificial Intelligence system DLV.

* The above listed static versions do not support I-DLV advanced features such as external atoms and SPARQL/SQL directives for importing/exporting data. Such features are supported by the following standalone versions:

	* Standalone executable of I-DLV for Linux x86-64 in which semantics of external atoms can be specified via Python3.5 functions (you need to install Python 3.5 to execute this version):
	
	* Standalone executable of I-DLV for Linux x86-64 in which semantics of external atoms can be specified via Python2.7 functions (you need to install Python 2.7 to execute this version):
		
Link
=============
		
* You can download the latest stable release of I-DLV `in the release section of github <https://github.com/DeMaCS-UNICAL/I-DLV/releases>`_.


Syntax
++++++++++++++

External Computations
============================

* I-DLV is intended as a tool to ease the interoperability of ASP with external sources of knowledge. To comply with this purpose, its input language, still supporting ASP-Core-2, has also been enriched with explicit constructs for demanding computations to external components as well as internally handled facilities for handling list terms, strings, etc.

* Further details are reported hereafter and in the related literature:

* Francesco Calimeri, Davide Fuscà, Simona Perri, Jessica Zangari. External Computations and Interoperability in the New DLV Grounder.

Python External Atoms
---------------------------

* Users can provide via python scripts with ``.py`` extension externally defined functionalities.

Syntax and Intuitive Semantics
___________________________________

* I-DLV supports the call to external sources of computations within ASP programs by means of external atoms in the rule bodies.

An external atom is of the form::

    &p (t_0 ,..., t_n ; u_0 ,..., u_m)

where::

    &p is an external predicate (the name must start with a & symbol)
    t_0, ..., t_n are input terms
    u_0, ..., u_m are output terms
    a semicolon symbol (";") separates input from output terms.

Note that each instance of an external predicate must appear with the same number of input and output terms throughout the program.

Intuitively, output terms are computed on the basis of the input ones, according to a semantics which is externally defined via Python scripts.

In particular, for each external predicate ``&p`` featuring ``n/m`` input and output terms, respectively, the user must define a Python function whose name is ``p``, and featuring ``n/m`` input and output parameters, respectively. The function has to be compliant with Python version 3.

External atoms can be both functional and relational, i.e., they can return a single tuple or a set of tuples, as output.

Notably, the functions defining the semantics of external atoms appearing in an input program can be defined in a unique python file or multiple ones, and the python files can be given to I-DLV without no particular command-line option.

Output Terms Mapping Policy
_______________________________

* Each value returned by the Python function defining an external predicate can be of one of these *types*: **numeric**, **boolean** or **string**. 

* These values are internally mapped to ground values accordingly to the following default policy: a integer returned value is mapped to a corresponding **numeric constant**; all other values are either associate to a **symbolic constant**, if the form is compatible according to the ASP-Core-2 syntax, otherwise are associated to a **string constant** in case of failure.

However, I-DLV allows the user to customize the mapping policy of a particular external predicate by means of a [directive] statement of the form::

    #external_predicate_conversion(&p,type:T_1, ..., T_N)

that specifies the sequence of conversion types for an external predicate &p featuring n output terms.

A conversion type can be:

    #. U_INT, the output value is converted to an unsigned integer;
    #. UT_INT, the output value is truncated to an unsigned integer;
    #. UR_INT, the output value is rounded to an unsigned integer;
    #. T_INT, the output value is truncated to an integer;
    #. R_INT, the output value is rounded to an integer;
    #. CONST, the output value is converted to a string without quotes;
    #. Q_CONST, the output value is converted to a string with quotes.

In cases ``1``, ``2``, ``3`` the value is mapped to a **numeric term**, in case ``4`` to a **symbolic constant**, while in case ``5`` in a **string constant**. Such directives have a global impact: they are applied to each occurrence of the external predicate they are referring to, and can be specified at any point within an ASP program.

We remark that, in previous I-DLV versions, this conversion policies were allowed by means of global annotations of the form::

    %@global_external_predicate_conversion(@predicate=&p,@type=@T_1, ..., @T_N)

with an identical meaning.

:ref:`ComputeSum`.

Native Directives
--------------------------

SQL Directives
________________

* We extend I-DLV with an ODBC interface by adding two new directives to import and export relations (predicates) from/to a DBMS. 
  These commands take a number of arguments providing the information needed for DBMS authentication, the relational predicate to import/export from/to the DBMS, and the name of the table in the DBMS.

Import SQL Directive
_____________________

* The ``#import_sql`` directive reads tuples from a specified table of a relational database and stores them as facts (EDB) with a predicate name ``p`` provided by the user. 
  The name of the imported atoms is set to ``p``, and defines a part of the EDB program. 
  Further EDB predicates can be added by providing input via text files. 
  Since I-DLV supports only integer and constant data types, the ``#import_sql`` commands take a parameter which specifies a type conversion for every column of the table.

The ``#import_sql`` command is of the form::

	#import_sql (databasename,"username","password","query", predname[, typeConv]).

where:

    #. `databasename` is the name of the ODBC DSN (Data Source Name);
    #. `username` defines the name of the user who connects to the database (the string must be enclosed by " ");
    #. `password` defines the password of that user (the string must be enclosed by " ");
    #. `query` is an SQL statement that constructs the table that will be imported (and must be quoted by " ");
    #. `predname` defines the name of the predicate that will be used;

``typeConv`` is optional and specifies the conversion for mapping DBMS data types to I-DLV data type; it provides a conversion for each column imported by the database.

The typeConv parameter is a string with the following syntax:``type: Conv [, Conv]``, where ``type:`` is a string constant and ``Conv`` is one of several conversion types:

    * U_INT: the column is converted to an unsigned integer;
    * UT_INT: the column is truncated to an unsigned integer;
    * UR_INT: the column is rounded to an unsigned integer;
    * T_INT: the column is truncated to an integer;
    * R_INT: the column is rounded to an integer;
    * CONST: the column is converted to a string without quotes;
    * Q_CONST: the column is converted to a string with quotes.

The number of the entries in the conversion list has to match the number of columns in the selected table. Strings converted as CONST should be valid ASP-Core-2 symbolic constants.

Export SQL Directives
________________________

* The ``#export_sql`` directive allows exporting the facts of a predicate in a DB. The ``#export_sql`` command comes in two variants. 

The first is of the form::

	#export_sql(databasename, "username", "password", predname, predarity, tablename).

The second variant adds another parameter, ``REPLACE where SQL-Condition``, which replaces the tuples in the table tablename for which SQL-Condition holds. 
It allows adding tuples to the table without creating conflicts whenever such tuples would violate any integrity constraint of the database (e.g. duplicate values for a key attribute)::

	#export_sql(databasename,"username","password", predname, predarity, tablename,"REPLACE where SQL-Condition").

where:

    #. `databasename` is the name of the database server;
    #. `username` is the name of the user who connects to the database (the string must be enclosed by " ");
    #. `password` provides the password of that user (the string must be enclosed by " ");
    #. `predname` and `predarity` defines the predicate that will be exported;
    #. `tablename` defines the name of the table;
    #. `REPLACE where SQL-Condition` contains the key words `REPLACE` and where followed by an SQL-Condition which indicates the tuples which shall be deleted from the relational table before the export takes place.

SPARQL Directives
__________________

* I-DLV supports two type different of directives::

	* The ``#import_local_sparql directive``: can be adopted to import data from a local RDF file.
	* The ``#import_remote_sparql directive``: can be adopted to import data from a remote RDF End Point.

.. note:: 
	
	both directives store the imported date as **facts (EDB)** of **a** predicate **p** provided by the user. 

* The behaviour of these directives is very similar to the ``#import_sql`` one. 

* The imported ground atoms define a part of the EDB program. Further EDB predicates can be added by providing input via text files. 
  Since I-DLV supports **only integer** and **constant data types**, the directives take a parameter which specifies a type conversion for every argument of the predicate p.

The ``#import_local_sparql`` command is of the form::

	#import_local_sparql (“rdf_file”, “quey”,predname, predarity, [, typeConv]).

While, the ``#import_remote_sparql`` command is of the form::

	#import_remote_sparql (“endpoint_url”, “quey”,predname, predarity, [, typeConv]).

where:

    rdf_file is the path of the RDF file (the string must be enclosed by “ ”);
    endpoint_url is the path of the RDF file (the string must be enclosed by “ ”);
    query is an SPARQL statement (and must be quoted by " ");
    predname and predarity define, respectively, the name and the arity of the predicate that will be used;

* ``typeConv`` is optional and specifies the conversion for mapping RDF data types to I-DLV data type. The conversion policy is the same of the ``#import_sql directive``.

* It is worth to point out that for the local import directive, rdf_file can be either a local or remote URL pointing to an RDF: in this latter case, the file is downloaded and treated as a local RDF file. 
  In this case the import is handled by a C++ module that builds the graph model in memory before performing the query.

* The remote import instead in intended to query a remote endpoint and hence building the graph is up to the remote server. 
  Moreover, this directive is implemented through a rewriting strategy that adopts external atoms and, thus python functions, to perform the query. 
  More in details, for each remote SPARQL import directive, an auxiliary rule of the following form is added to the input program::

	predname(X_0, ...,X_N) :- &sparqlEndPoint(“endpnt_url”,“query”;X_0, ...,X_N).

* The head atom has ``predname`` as name, and contains a number of variables that corresponds to predarity, while the body contains an external atom ``&sparqlEndPoint`` in charge of performing the remote query.

Regards the efficiency, we observed that, typically, even if the remote directive involves the invocation of external python code, it might be convenient in case of large datasets with an intrinsic complex graph model.

Default External Atoms and List Terms
-------------------------------------------

* From version 1.1.5, I-DLV provides a wide set of built-in functions with predefined semantics: their syntax is in line with that of python external atoms and as in the case of python external atoms, default external atoms are completely evaluated at grounding time. 
  Default external atoms provides ready-to-use features to manage integers, strings and list terms. 
  They can be used as assignments, i.e., to generate new values for output variables, or in comparisons, i.e., to check if the specified conditions hold. 
  However, since all these functionalities, described below, are intenally implemented within the system, for their usage it is not needed any python script.

* An default external literal is either not e or e, where e is a default external atom, and the symbol not represents default negation.

* Nevertheless, users can redefine the semantics of such default external atoms. 
  In particular, a built-in function can be redefined by providing as input to I-DLV a python script with .py extension, specifying a function with the same name and parameters; 
  in this case, the user externally provided sematics are used in place of the default internal one. 
  In this case, one can still force the system to adopt the default semantics via the command line option ``--default-external-atom``.

Arithmetic Facilities
__________________________

.. list-table:: Arithmetic
	:widths: 10 20 20 20
	:header-rows: 1
	
	* - Atom
	  - Semantics in Assignment               	      
	  - Semantics in Comparison                            
	  - Constraints

	* - ``&abs(X;Z)``        
	  - assigns the absolute value of X to Z         
	  - true iff Z=|X| holds                               
	  - X must be an integer
	
	* - ``&int(X,Y;Z)``      
	  - generates all Z integers s.t. X<=Z<=Y        
	  - true iff X<=Z<=Y holds                             
	  - X and Y must be integers and X<=Y
	   
	* - ``&mod(X,Y;Z)¹``     
	  - assigns the result of X%Y to Z               
	  - true iff Z=X%Y holds                               
	  - X and Y must be integers and Y!=0
	  
	* - ``&rand(X,Y;Z)``     
	  - assigns a random number to Z s.t. X<=Z<=Y    
	  - true iff Z is equal to the selected random number  
	  - X and Y must be integers and X<=Y

	* - ``&sum(X,Y;Z)²``     
	  - assigns the result of X+Y to Z               
	  - true iff Z=X+Y holds                               
	  - X and Y must be integers

``¹`` It can also be expressed using a built-in atom of the form: ``Z=X\Y``. Notice that in this case Z must be bound in order to make the rule safe, i.e., it must be contained within a positive literal or involved within an assignment.

``²`` It can also be expressed using a standard built-in atom of the form: ``Z=X+Y``.

* :ref:`Arithmetic`.
	
Strings Facilities
___________________

.. list-table:: Strings
	:widths: 10 20 20 20
	:header-rows: 1
	
	* - Atom
	  - Behaviour in Assignment               	      
	  - Behaviour in Comparison                            
	  - Constraints

	* - ``&append_str(X,Y;Z)``        
	  - appends Y to X, the resulting string is assigned to Z         
	  - true iff Z is equal to the string obtained appending Y to X                          
	  - X, Y and Z can be numeric, symbolic or string constants
	
	* - ``&length_str(X;Z)¹``      
	  - assigns the length of X to Z        
	  - true iff length(X)<=Z holds                          
	  - X can be numeric, symbolic or string constant. Z must be an integer
	   
	* - ``&member_str(X,Y;)``     
	  - -            
	  - true iff Y contains X                            
	  - X and Y can be numeric, symbolic or string constants
	  
	* - ``&reverse_str(X;Z)¹``     
	  - computes the reverse of X, the resulting string is assigned to Z   
	  - true iff Z is equal to the reverse of X 
	  - X and Z can be numeric, symbolic or string constants

	* - ``&sub_str(X,Y,W;Z)``     
	  - generates a substring of W starting from X to Y, the resulting string is assigned to Z              
	  - true iff Z is equal to the substring of W starting from X to Y                              
	  - X and Y must be integers s.t..  W must be either a symbolic or string constant, Z must be a string constant

``¹`` it is able to deal with strings containing special characters performing an internal conversion from the UTF-8 encoding to the UTF-16 one. The command line option ``--no-string-conversion`` can be used to disable this conversion.

* :ref:`String`.

List Terms
____________

* List terms belong to the class of `complex terms`. In particular, a **list term** is a sequence of any size composed both by "normal" and complex terms and can be of two forms:

* ``[ t1, . . . , tn ]``,where ``t1, . . . , tn`` are terms;
* ``[ h | t ]``, where ``h`` (the head of the list) is a term, and ``t`` (the tail of the list) is a list term;

* From **version 1.1.5**, I-DLV fully supports these list terms and provides several default external atoms designed to facilitate their usage.

.. list-table:: List
	:widths: 10 20 20 20
	:header-rows: 1
	
	* - Atom
	  - Behaviour in Assignment           	      
	  - Behaviour in Comparison 	                          
	  - Constraints

	* - ``&append(L1,L2;LR)``        
	  - appends L2 to L1, the resulting list is assigned to LR        
	  - true iff LR is equal to the list obtained appending L2 to L1                         
	  - L1 and L2 must be list terms, LR must be either a variable or a list term
	
	* - ``&delNth(L,P;LR)``      
	  - deletes the element at position P in L, the resulting list is assigned to LR       
	  - true iff LR is equal to the list obtained deleting the P-th element from L                        
	  - L must be a list terms, LR must be either a variable or a list term and P must be an integer s.t. 0<P
	   
	* - ``&flatten(L;LR)``     
	  - flattens L and assigns the resulting list to LR           
	  - true iff LR is equal to the list obtained by flattening the list term L                           
	  - L must be a list term, LR must be either a variable or a list term
	  
	* - ``&head(L;E)``     
	  - assigns the head of L to E   
	  - true iff E is equal to the head of L
	  - L must be a list term

	* - ``&insLast(L,E;LR)``     
	  - appends E to L, the resulting list is assigned to LR            
	  - true iff LR is equal to the list obtained appending E to L        
	  - L must be a list term, LR must be either a variable or a list term

	* - ``&insNth(L,E,P;LR)``
	  - inserts E at position P of L,the resulting list is assigned to LR         	      
	  - is true iff LR is a list obtained by inserting the term E into L at position P	                          
	  - L must be a list term, LR must be either a variable or a list term, P must be an integer s.t P>O

	* - ``&last(L;E)``
	  - assigns the last element of L to E           	      
	  - true iff E is equal to the last element of L	                          
	  - L must be a list term
	
	* - ``&length(L;Z)``
	  - assigns the length of L to Z      	      
	  - true iff length(L)>0 holds	                          
	  - L must be a list term, Z must be either a variable or an integer

	* - ``&member(E,L;)``
	  - -          	      
	  - true iff L contains the term E	                          
	  - L must be a list term

	* - ``&memberNth(L,P;E)``
	  - assigns to E the term contained at position P of L           	      
	  - true iff E is equal to the element at position P in L	                          
	  - L must be a list terms and P must be an integer s.t. P>0

	* - ``&reverse(L;LR)``
	  - computes the reverse of L, the reversed string is assigned to LR           	      
	  - true iff LR is equal to the reverse of L	                          
	  - L must be a list term, LR must be either a variable or a list term

	* - ``&reverse_r(L;LR)``
	  - computes the reverse of L and of all nested list terms, the reversed L is assigned to LR        	      
	  - true iff LR is equal to the list obtained by reversing L and all nested list terms 	                          
	  - L must be a list term, LR must be either a variable or a list term

	* - ``&delete(E,L;LR)``
	  - delete all occurrences of E in L, the resulting list is assigned to LR          	      
	  - true iff LR is equal to the list obtained by removing all occurrences of E in L	                          
	  - L must be a list term, LR must be either a variable or a list term

	* - ``&delete_r(E,L;LR)``
	  - delete all occurrences of E in L and in all the nested list, the resulting list is assigned to LR
	  - true iff LR is equal to the list obtained by removing all occurrences of E in L and in all nested list terms 	                          
	  - L must be list term, LR must be either a variable or a list term

	* - ``&subList(L1,L2;)``
	  - -           	      
	  - true iff L1 is a subL of L2	                          
	  - L1 and L2 must be list terms

	* - ``&tail(L;E)``
	  - assigns the tail of L to E         	      
	  - true iff E is equal to the tail of L	                          
	  - L must be a list term
	  
* :ref:`List`.

Example
++++++++++++++

External Computations
===========================

.. _ComputeSum:

Example Compute Sum
-------------------------

As an example, let us consider the following program, that makes use of an external predicate with two input and one output terms::

	compute_sum(X,Y,Z) :- &sum(X,Y;Z), number(X), number(Y), X<=Y. 
	number(1..3).

A program containing this rule must come along with the proper definition of sum Python function, as, for instance::

	def sum(X,Y):
	    return X+Y

When grounding the program I-DLV will invoke the python function sum for each couple of numbers, obtaining::

	compute_sum(1,1,2) :- sum(1,1;2), number(1), number(1).
	compute_sum(1,2,3) :- sum(1,2;3), number(1), number(2).
	compute_sum(1,3,4) :- sum(1,3;4), number(1), number(3).
	compute_sum(2,2,4) :- sum(2,2;4), number(2), number(2).
	compute_sum(2,3,5) :- sum(2,3;5), number(2), number(3).
	compute_sum(3,3,6) :- sum(3,3;6), number(3), number(3).

By applying the simplification process, since the program is a stratified and non-disjunctive, I-DLV will determine its unique answer set.

Hence, supposing that the ASP program is contained in a file called ``file.asp``, while the python function sum is defined in file called ``python_file.py`` by invoking::

	./idlv file.asp python_file.py 

It will be obtained as output from I-DLV::

	compute_sum(1,1,2). 
	compute_sum(1,2,3). 
	compute_sum(1,3,4).
	compute_sum(2,2,4).
	compute_sum(2,3,5).
	compute_sum(3,3,6).

Notably, if in the program the following annotation is added::

	@global_external_predicate_conversion(@predicate=&sum,@type=@Q_CONST).

Then, the unique answer sets of the program would be, instead::

	compute_sum(1,1,"2"). 
	compute_sum(1,2,"3"). 
	compute_sum(1,3,"4").
	compute_sum(2,2,"4"). 
	compute_sum(2,3,"5"). 
	compute_sum(3,3,"6").

.. _Arithmetic:

Example Arithmetic
-------------------------

Input program::

	number(Z) :- &int(-3,3;Z).
	even(Z) :- number(Z), &mod(Z,2;0).
	odd(Z) :- number(Z), not &mod(Z,2;0).

Instantiation output::

	number(-3). number(-2). number(-1). 
	number(0). number(1). number(2). 
	number(3).
	even(-2). even(0). even(2).
	odd(-3). odd(-1). 
	odd(1). odd(3).

.. _String:

Example String
-----------------------

Input facts::

	str(123). 
	str("23"). 
	str(abc). 
	str("xyz").

Input program::

	sub_str(W,Z) :- str(W), &length_str(W;Z1), Z1>=3, &to_qstr(W;W1), &sub_str(2,3,W1;Z).
	rev_str(X,Z) :- str(X), &reverse_str(X;Z).

Instantiation output::

	str(123). str("23"). str(abc). str("xyz").
	sub_str(123,"23"). sub_str(abc,"bc"). sub_str("xyz","yz").
	rev_str(123,"321"). rev_str("23","32"). rev_str(abc,cba). rev_str("xyz","zyx").

.. _List:

Example List
------------------

Input facts::

	list([1,2,3]). list([[1,2]|[3,4,[5,6]]]).

Input program::

	flattened_list(Z):-list(L),&flatten(L;Z).
	reverse(Z):-list(L),&reverse(L;Z).
	recursive_reverse(Z):-list(L),&reverse_r(L;Z).
	delete_element(Z):-list(L),&delete(2,L;Z).
	recursive_delete_element(Z):-list(L),&delete_r(2,L;Z).
	head(Z):-list(L),&head(L;Z).
	tail(Z):-list(L),&tail(L;Z).

Instantiation output::

	list([1,2,3]). list([[1,2],3,4,[5,6]]).
	flattened_list([1,2,3]). flattened_list([1,2,3,4,5,6]).
	reverse([3,2,1]). reverse([[5,6],4,3,[1,2]]).
	recursive_reverse([3,2,1]). recursive_reverse([[6,5],4,3,[2,1]]).
	delete_element([1,3]). delete_element([[1,2],3,4,[5,6]]).
	recursive_delete_element([1,3]). recursive_delete_element([[1],3,4,[5,6]]).
	head(1). head([1,2]).  
	tail([2,3]). tail([3,4,[5,6]]).
