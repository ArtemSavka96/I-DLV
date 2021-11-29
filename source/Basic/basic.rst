.. toctree::
   :maxdepth: 3
   :caption: Basic Version   

   basic

About
+++++++++++++++++++++++++

* The basic version of I-DLV supports `ASP-Core-2`_ plus some addictional lanuage extensions suuch as list terms, default external literals and annotations.

* The basic version of I-DLV:

	* **Linux**: Static executable of I-DLV for Linux x86-64.

	* **Windows**: Static executable of I-DLV for Windows 64bit.
		
Download
========================
		
* You can download the latest stable release of I-DLV `in the release section of github <https://github.com/DeMaCS-UNICAL/I-DLV/releases>`_.
	
How to execute
================================

* After downloading the I-DLV, we have to make the file **executable** and depending on the operating system:

	* On Windows without WSL:
		
		#. Rename the file idlv.exe
		
	* On Linux or Windows with WSL:
	
		#. Open a terminal on Linux or the WSL terminal
		#. Move using the `cd` command to the folder in which we downloaded I-DLV
		#. Rename the idlv using the `mv` command::
			
			mv file_download idvl
			
		#. Execute the following command on the terminal::
		
			chmod + xX idlv
			
	* On MacOS you need to perform a procedure similar to Linux ``3.`` and ``4.``

**Execute I-DLV**

	* On Linux,MacOs or WSL terminal::
		
		./idlv file.txt
		
	* On Windows, from the command prompt::
		
		idlv.exe file.txt

* I-DLV can interoperate with the state-of-the-art solvers:
	* `wasp <https://github.com/alviano/wasp/>`_.
	* `clasp <https://sourceforge.net/projects/potassco/files/clasp>`_.
	
* Indeed, by default I-DLV output is produced in a numeric format compliant with the mentioned solvers.

* In order to interoperate with a solver type::
		
		./idlv [filename [filename [...]]] | ./solver_executable
	
* In order to obtain the ground program in textual format type::
	
		./idlv --t [filename [filename [...]]]


How to cite I-DLV
============================

* Francesco Calimeri, Davide Fuscà, Simona Perri, Jessica Zangari: `I-DLV: The New Intelligent Grounder of dlv. AI*IA 2016: 192-207 <https://www.researchgate.net/publication/316352879_I-DLV_The_new_intelligent_grounder_of_DLV>`_

* Francesco Calimeri, Davide Fuscà, Simona Perri, Jessica Zangari: `I-DLV: The new intelligent grounder of DLV. Intelligenza Artificiale 11(1): 5-20 (2017) <https://www.researchgate.net/publication/316352879_I-DLV_The_new_intelligent_grounder_of_DLV>`_

.. _BasicSyntax:

Syntax
+++++++++++++++

Basic Language
===================

Terms
*****************

* Terms are either constants or variables
* Constants can be either symbolic constants (strings starting with some lowercase letter), string constants (quoted strings) or integers::

	Ex.: pippo, “this is a string constant”, 123, …
	
* Variables are denoted by strings starting with some uppercase letter::

	Ex.: X, Pippo, THIS_IS_A_VARIABLE, White, …

Atoms and Literal
******************

* A predicate atom has form `p(t1,…,tn )`, where `p` is a predicate name, `t1 ,…, tn` are terms, and `n≥0` is the arity of the predicate atom. 

* A predicate atom p() of arity 0 is likewise represented by its predicate name p without parentheses::

	Ex.: p(X,Y) - next(1,2) - q - i_am_an_atom(1,2,a,B,X)

* An atom can be negated by means of `not`::

	Ex: not a, not p(X), …
	
* A literal is an atom or a negated atom. In the first case it is said to be positive, while in the second it is said to be negative.


Facts
*****************


* A ground rule with an empty body is called a fact.
* A fact is therefore a rule with a True body (an empty conjunction is true by definition).
* The implication symbol is omitted for facts::

	parent(eugenio, peppe) :- true.
	parent(mario, ciccio) :- true.
	

equivalently written by::

	parent(eugenio, peppe).
	parent(mario, ciccio).

.. caution::

	Facts must always be true in the program answer!

A program with comments
****************************

:: 

          %This line is a comment.
          weight(apple,100,gram). %Here is a new comment, it ends in this row.
        

Costraint and Functions
==========================

Integrity Constraints
**********************

* Constraints in our framework specify conditions which must not become true in any model. 
	In other words, constraints are formulations of possible inconsistencies. 
	This mechanism is very useful in connection with disjunctive rules. 
	The disjunctive rules serve as generators for different models and the constraints are used to select only the desired ones.

* The syntax of constraints is simple:
			
	They look like rules without heads. 
	As with rules, constraints must meet the safety requirements.

A very well-known problem is to find a coloring of a graph, such that no two adjacent nodes (i.e. two nodes which are connected by an arc) have the same color.

To formulate this problem in I-DLV, let us reconsider the coloring example from above::

	  node(X) :- arc(X,Y).
	  node(Y) :- arc(X,Y).

	  color(X,red) | color(X,green) | color(X,blue) :- node(X).
	

The models of this program (together with a database describing the graph) correspond to all possible colorings. We only need to add a constraint which discards colorings where two adjacent nodes have the same color::

	  :- arc(X,Y), color(X,C), color(Y,C), X<Y.
	

Assuming the combined code resides in 3col, the output then is::

	  {node(1), node(2), node(3), node(4), color(1,red), color(2,green), color(3,red), color(4,red)}

	  [... and so on ...]
	  
	  {node(1), node(2), node(3), node(4), color(1,blue), color(2,green), color(3,blue), color(4,red)}
	  {node(1), node(2), node(3), node(4), color(1,blue), color(2,green), color(3,blue), color(4,blue)}
	

Note that this program yields 24 models, while the previous coloring example led to 81 models. 
 
Constraints and negation
*************************

Of course, also negation can be used in constraints::

            a | b.
            :- not a.
          

`{a}` is a model of this program, but `{b}` is not, since the constraint would be violated for `{b}`.

As a comparison, a constraint with true negation:

            a | b.
            :- -a.
          

Both `{a}` and `{b}` are models, since `-a` is not contained in any of these models.  

Safety
*************************

* A rule `r` is safe: *if each variable in the head*, and *each variable in a negative literal*, and *each variable in a comparison operator (<,<=, etc.)*
  also appears in a standard positive literal. 

In other words, all variables must appear at least once in the positive body.

**Only safe rules are allowed.**

Ex.: The following rules are unsafe::

		s(X) :- a.
		s(Y) :- b(Y), not r(X).
		s(X) :- not r(X).
		s(Y) :- b(Y), X<Y.

In each case, an infinity of x’s can satisfy the rule, even if `r` is a finite relation.

Weak Constraints
*************************

Syntax::

	:~ B [ W@P, T 1 …T n ].

**Satisfy B if possible; if not, pay W at priority P for each**
**distinct tuple of terms** ``T 1 …T n`` .

projecting

:: 

	:~ p(X,Y). [ 1@1 ]

is equivalent to::

	:~ p(X,Y). [ 1@1, X,Y]

while::

	:~ p(X,Y). [ 1@1, X]

**is different**, and corresponds to::

	:~ q(X). [1@1]
	q(X) :- p(X,Y).

* Example

	A::
	
		:~ p(X,Y). [ 1@1, X] 
		equivalent to 
		:~ q(X). [1@1]
		q(X) :- p(X,Y).
		
	B::
	
		:~ p(X,Y). [ 1@1, X,Y] 
		equivalent to 
		:~ p(X,Y) [1@1]

With facts::

	p(1,2). p(1,3).
	
Costs::
	
	A: costs [1@1]. 
	B: costs [2@1].


Aggregate function
*************************

#. ``#count``::

	nodeNumber(N):-#count{ X: node(X)}=N.	
	
#. ``#sum``::

	:-node(X),#sum{SumEdgeValue,Node: node(Node), edge(X,Node,SumEdgeValue)}=0.
	
#. ``#min``::

	minAge(N):-#min{X,Name: person(Name,X)}=N.		
	
#. ``#max``::

	:-#max{X,Name: person(Name,X)}=Max, minAge(Min), Max-Min=0.


Modelling and Programming Techniques
======================================

Recursion
*************************

* In mathematics and computer science, a recursive definition, or inductive definition, is used to define the elements in a set in terms of other elements in the set. 
  Some examples of recursively-definable objects include factorials, natural numbers, Fibonacci numbers, and the Cantor ternary set.


For example of Recursion:

* :ref:`Recursion`. 

Transitive Closure
*************************

* In computer science, the concept of transitive closure can be thought of as constructing a data structure that makes it possible to answer reachability questions. 
  That is, can one get **from** ``node a`` **to** ``node d`` in one or more hops? A binary relation tells you only that ``node a`` is connected to ``node b``, and that ``node b`` is connected to ``node c``, etc. 
  After the transitive closure is constructed, in an ``O(1)`` operation one may determine that ``node d`` **is reachable from** ``node a``. 
  The data structure is typically stored as a matrix, so if ``matrix[1][4] = 1``, then it is the case that ``node 1`` can reach ``node 4`` through one or more hops.

The transitive closure of the adjacency relation of a directed acyclic graph (DAG) is the reachability relation of the DAG and a strict partial order.

For example of Transitive Clousure:

* :ref:`TransitiveClousure`.


The GCO
*************************

**Guess/Check/Optimize**

* Many problems can be solved in a natural manner by using this
  declarative programming technique.

* The power of disjunctive rules allows to express problems which are
  more complex than NP, and the (optional) separation of a fixed, 
  non-ground program from an input database allows to do so in a uniform way over varying instances.

Generalization of the Guess and Check method
to express optimization problems
A program is made of 3 Modules:

* ``[Guessing Part]`` defines the search space
* ``[Checking Part]`` checks solution admissibility
* ``[Optimizing Part]`` specifies a preference criterion (by means of weak constraints)

Guess
__________________

* Defines the search space.
* Such that answer sets represent “solution candidates”.

* Example::

	inPath(X,Y) | outPath(X,Y) :- arc(X,Y,Z).
	
Check
__________________

* Filter the solution candidates.
* Represent the admissible solutions.

* Example::

	:- inPath(X,Y), inPath(X,Y1), Y < Y1.
	:- inPath(X,Y), inPath(X1,Y), X < X1.
	:- node(X), not reached(X).

Optimize
__________________

* That specifies a quantitative cost evaluation of solutions.

* Example::

	:~ inPath(X,Y), arc(X,Y,C). [C@1, X,Y,C]

For example of GCO or GC:

* :ref:`HamP`.

* :ref:`MST`.

* :ref:`3Col`.

Default External Atoms and List Terms
======================================

* From version 1.1.5, I-DLV provides a wide set of built-in functions with predefined semantics: their syntax is in line with that of `Python`_ external atoms and as in the case of `Python`_ external atoms, default external atoms are completely evaluated at grounding time. 
  Default external atoms provides ready-to-use features to manage integers, strings and list terms. 
  They can be used as assignments, i.e., to generate new values for output variables, or in comparisons, i.e., to check if the specified conditions hold. 
  However, since all these functionalities, described below, are intenally implemented within the system, for their usage it is not needed any `Python`_ script.

* An default external literal is either not e or e, where e is a default external atom, and the symbol not represents default negation.

* Nevertheless, users can redefine the semantics of such default external atoms. 
  In particular, a built-in function can be redefined by providing as input to I-DLV a `Python`_ script with .py extension, specifying a function with the same name and parameters; 
  in this case, the user externally provided sematics are used in place of the default internal one. 
  In this case, one can still force the system to adopt the default semantics via the command line option ``--default-external-atom``.
	
List
*************************

* A list is a binary function denoted with a special syntax: ``[H|T]``

* Where the first argument **«H»** is a term, called the
  head of the list, and the second argument **«T»** is a list.

In addition, a list can be represented by explicitly
listing its elements.

	[ a, b, c ] = [ a | [ b, c ] ] = [ a | [ b | [ c ] ] ] = [a|[b|[c|[]]]]

* LIST TERMS

	A list term can be of the two forms::
	
		* [ t1, . . . , tn ], where t1, . . . , tn are terms;
		* [ h | t ], where h (the head of the list) is a term, and t (the tail of the list) is a list term.

* Examples::

	* The term [ a,d,a ] in the atom palindromic([a,d,a])
	* [ jan,feb,mar ]
	* [jan | [ feb,mar,apr,may,jun ] ]
	* [ [jan,31] | [ [ feb,28 ], [ mar,31 ], [ apr,30 ], [ may,31], [ jun,30 ] ] ].

I-DLV (and DLV2) comes with a rich library of built-in predicates for list manipulation.

A built-in atom is of the form::
	
	&p(t 0 ,.., t n ; u 0 ,…, u m )

where n,m >= 0

* ``t 0 ,.., t n`` are input terms, and are separated from the output terms ``u 0 ,…, u m`` by a semicolon (“;”);
* an input/output term can be either a constant or a variable.

Intuitively, output terms are computed on the basis of the input
ones, according to a semantics which is defined "a priori" for
each predicate, as reported next.

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

Arithmetic Facilities
*************************

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
**********************

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

Annotations
*************************

* I-DLV introduces a new special feature for facilitating system customization and tuning: annotations of ASP code. 
  I-DLV annotations allow to give explicit directions on the internal grounding process, at a more fine-grained level with respect to the command-line options: 
  they **annotate** the ASP code in a Java-like fashion while embedded in comments, so that the resulting programs can still be given as input to other ASP systems, without any modification.

* Syntactically, all annotation start with the prefix ``%@`` and end with a dot (``.``). 
  Currently, I-DLV supports annotations for customizing two of the major aspects of the grounding process, body ordering and indexing as well as further optimizations intervening in its grounding process.

A specific body ordering strategy can be explicitly requested for any rule, simply preceding it with the line::

	%@rule_ordering(@value=Ordering_Type).

where Ordering_Type is a number representing an ordering strategy. In addition, it is possible to specify a particular partial order among atoms, no matter the employed ordering strategy, by means of before and after directives. For instance, in the next example I-DLV is forced to always put literals ``a(X,Y) and X = #count{Z : c(Z)}}`` before literal ``f(X,Y )`` , whatever the order chosen::

 	%@rule_partial_order(@before={a(X,Y),X=#count{Z:c(Z)}},@after={f(X,Y)}).

As for indexing, directives on a per-atom basis can be given; the next annotation, for instance, request that, in the subsequent rule, atom ``a(X,Y,Z)`` is indexed, if possible, with a double-index the first and third arguments::

  	%@rule_atom_indexed(@atom=a(X,Y,Z),@arguments={0,2}).

The projection rewriting can be customized for any rule, by preceding it with the line::

  	%@rule_projection(n).

where ``n`` can either ``0``, ``1`` or ``2``, as for the command-line projection option described above.

The rewriting arithmetic terms, disabled by default, can be enabled for any rule, by specifying before it the following annotation::

  	%@rule_rewrite_arith().

Similarly, the aligning substitution technique disabled by default, can be enabled for a specific rule by preceding it with::

  	%@rule_align_substitutions().

while, for enabling the look-ahead technique, the annotation to use is the following::

  	%@rule_look_ahead().

* Multiple preferences can be expressed via different annotations; in case of conflicts, priority is given to the first. 
  In addition, preferences can also be specified at a **global** scope, by replacing the rule directive with the global one. 
  While a rule annotation must precede the intended rule, global annotations can appear on any line in the input program. 
  Both global and rule annotations can be expressed in the same program; in case of overlap on a particular rule/setting, priority is given to the rule ones.


.. _BasicExample:

Example
++++++++++++++++

.. _Recursion:

Same generation
============================

* Given a parent-child relationship, represented by an acyclic directed graph, 
  we want to find all pairs of persons belonging to the same generation. 

* Two persons are of the same generation, if either they are siblings, 
  or they are children of two persons of the same generation. 

* If input is encoded by a relation ``parent(A,B)``, 
  where a fact ``parent(a, b)`` states that ``a`` is ``a parent of b``. 

* The solution can be encoded by the following program, which
  computes a relation ``ancestor(A,B)`` containing all facts such that A is of the same generation as B:

If we want to define the relation of arbitrary ancestors rather than grandparents, we make use of recursion::

	ancestor(A,B) :- parent(A,B).
	ancestor(A,C) :- ancestor(A,B), ancestor(B,C).

An equivalent representation is::

	ancestor(A,B) :- parent(A,B).
	ancestor(A,C) :- ancestor(A,B), parent(B,C).

.. _TransitiveClousure:

Reachability
============================
 
* Given a finite directed graph ``G = (N,A)``, 
  we want to compute all pairs of nodes ``(a, b) ∈ N × N`` such that ``b`` is reachable from ``a`` through a not empty sequence of arcs in A. 

* In other words, the problem amounts to computing the transitive closure of the relation A.
  Suppose we are representing a graph by a relation ``edge(X,Y)``.

I want to express the query: Find all nodes reachable from the others.

:: 

	path(X,Y) :- edge(X,Y).
	path(X,Y) :- path(X,Z), path(Z,Y).


.. _HamP:

Hamiltonian Path
============================

* Given a directed graph ``G = (N,A)`` and ``a`` node ``a ∈ N`` of this graph, 
  does there exist a path in G starts from ``a`` and passing through ``each node in N`` exactly once?

* If the graph G is specified by means of facts over predicates ``node (unary)`` and ``arc (binary)``, 
  and the starting node a is specified by the predicate ``start (unary)``, then, the following **GC** program Php encodes a solution to the problem::

	% Guessing Part (3 rules)
	
	% The two disjunctive rules guess a subset S of the arcs to be in the path.
	inPath(X, Y) | outPath(X, Y) :– start(X), arc(X, Y).
	inPath(X, Y) | outPath(X, Y) :– reached(X), arc(X, Y).

	% Transitive property, or that all node inPath are reachable from start node.
	reached(X) :– inPath(Y, X).

	% Checking Part (3 rules)

	% There must not be two arcs ending in the same node.
	:– inPath(X, Y), inPath(X, Y1), Y != Y1.

	% There must not be two arcs starting at the same node.	
	:– inPath(X, Y), inPath(X1, Y), X != X1.

	% All nodes in the graph are reached from the starting node.
	:– node(X), not reached(X), not start(X).

.. _MST:

Minimum Spanning Tree
============================

* Given a weighted graph by means of ``edge(Node1,Node2,Cost)``,
  and ``node(N)``, compute a tree that starts at a root node, 
  spans that graph, and has minimum cost, then, the following **GCO** program Pmst encodes a solution to the problem::

	% Guess the edges that are part of the tree:
	inTree(X,Y) | outTree(X,Y) :- edge(X,Y,C).

	% Check that we are really dealing with a tree!
	:- root(X), inTree(Y,X).
	:- inTree(X,Y), inTree(X1,Y), X != X1.
	
	% And the tree is connected
	:- node(X), not reached(X).

	% Minimize the cost of the tree
	:~ inTree(X,Y), edge(X,Y,C). [C@1, X,Y,C]

	reached(X) :- root(X).
	reached(X) :- reached(Y), inTree(Y,X).
	
.. _3Col:

3-colorability
============================

* Given a graph ``G = (N,A)`` and ``a`` node ``a ∈ N`` of this graph, it's a 3-colorability?

* Now, we can see three different and equivalent approaches:

First Mode
*************************

With **GC**, we extrapolate the basic rules for solving the Problem P3c::

	% Guessing Part
	% Guess the color of the node.
	col(X, red) | col(X, blue) | col(X, green) :- node(X).
	
	% Checking Part
	% No adjacent nodes with the same color.
	:- edge(X, Y), col(X,C), col(Y,C).

Second Mode
*************************

With **GC**, we can solve this problem P3c with one disjunctive rule for color::
 
	% Guessing Part
	% The three disjunctive rules guess a color C to node X. 
	col(X, red) | ncol(X, red) :- node(X).
	col(X, yellow) | ncol(X, yellow) :- node(X).
	col(X, green) | ncol(X, green) :- node(X).

	% Checking Part
	% No adjacent nodes with the same color.
	:- edge(X, Y), col(X,C), col(Y,C).
	
Third Mode
*************************

In fine, with **GC**, we use a different approach for solving the Problem P3c::
 
	% Guessing Part
	% Given a node X and a color C, color X with C or not
	col(X, C) | ncol(X, C) :- node(X), color(C).

	% Checking Part
	% No adjacent nodes with the same color
	:- edge(X, Y), col(X,C), col(Y,C).

	% All nodes must be colored
	colored(X):- col(X,Y).
	:- node(X), not colored(X).

	% Only one color per node
	:- col(X, C1), col(X,C2), C1!=C2.

.. _Arithmetic:

Arithmetic
============================

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

String
============================

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

List
============================

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

.. _Publication: https://www.researchgate.net/publication/316352879_I-DLV_The_new_intelligent_grounder_of_DLV
.. _ASP-Core-2: https://arxiv.org/abs/1911.04326 
.. _DLV: https://dlv.demacs.unical.it   
.. _Python: https://www.python.org