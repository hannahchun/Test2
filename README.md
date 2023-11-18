##### 22000662 Chun Hye Sun
# HW4 Implement RFAE in your favorite langauge

## How to Compile and Run

### In Eclipse
File → Import → Existing Projects into Workspace → Next → Select archive file → Browse → Choose HW4_RFAE.zip → Finish -> Run -> Run Configurations -> Select Arguments tab

#### Enable only parser
ex1. -p "{with {fac {fun {n} {if {= n 0} 1 {* n {fac {- n 1}}}}}} {fac 4}}"

#### Enable interpreter
ex1. "{with {fac {fun {n} {if {= n 0} 1 {* n {fac {- n 1}}}}}} {fac 4}"

### In Terminal
1. Move to src directory and check if bin directory exists. bin directory contains .class files.
2. Compile
javac -d [directory containing .class files] [class path where the main method exists] [.java file_that_has_main_method] [class path where other java files exists] [all .java files under that class path]\
ex1. **javac -d bin edu/handong/csee/plt/Main.java** **edu/handong/csee/plt/ast/*.java** **edu/handong/csee/plt/defsub/*.java** **edu/handong/csee/plt/retval/*.java**
3. Run
java -cp [directory containing .class files] [class path where the main method exists] [class name_that_has_main_method] [<option.>] ["<RFAE.>"]

* <option.> : not always required
* <RFAE.> : a concrete syntax ready to be converted into RFAE type

##### Enable only parser
set <option.> to '-p'.\
ex1. java -cp bin edu/handong/csee/plt/Main -p "{with {fac {fun {n} {if {= n 0} 1 {* n {fac {- n 1}}}}}} {fac 4}}”

##### Enable interpreter
do not set <option.>.\
ex1. java -cp bin edu/handong/csee/plt/Main "{with {fac {fun {n} {if {= n 0} 1 {* n {fac {- n 1}}}}}} {fac 4}}”

## Java files information
* **'edu/handong/csee/plt' package**
  * Main.java\
    'main' method exists here
  * Parser.java\
    concrete syntax -> abstract syntax
  * Interpreter.java\
    abstract syntax -> corresponding result (RFAEValue type)
  * Lookup.java\
    For the given symbol, find the corresponding value in the DefrdSub cache. If there is no symbol return free identifier error.

* **'edu/handong/csee/plt/ast' package**
  * Add.java
  * App.java
  * AST.java
  * Eqop.java
  * Fun.java
  * Id.java
  * Ifexp.java
  * Mul.java
  * Num.java
  * Orop.java
  * Sub.java
  
* **'edu/handong/csee/plt/defsub' package**
  * ASub.java
  * DefrdSub.java
  * MtSub.java

* **'edu/handong/csee/plt/retval' package**
  * ClosureV.java
  * RFAEValue.java
  * NumV.java

## RFAE: BNF
<RFAE.> :: = <num.>\
            | {+ <RFAE.> <RFAE.>}\
            | {- <RFAE.> <RFAE.>}\
            | {* <RFAE.> <RFAE.>}\
            | <id.>\
            | {fun {<id.> <RFAE.>}}\
            | {<RFAE.> <RFAE.>}\
            | {ifexp <RFAE.> <RFAE.> <RFAE.>}\
            | {orop <RFAE.> <RFAE.>}\
            | {eqop <RFAE.> <RFAE.>}

##  Testcase

**recursion examples**

ex1. {with {fac {fun {n} {if {= n 0} 1 {* n {fac {- n 1}}}}}} {fac 4}}

Desugared into\
{with {mk-rec {fun {body-proc} {with {fX {fun {fY} {with {f {fun {x} {{fY fY} x}}} {body-proc f}}}} {fX fX}}}} {with {fac {mk-rec {fun {fac} {fun {n} {if {= n 0} 1 {* n {fac {- n 1}}}}}}}} {fac 4}}}

Parsing Result\
(app (fun 'mk-rec (app (fun 'fac (app (id 'fac) (num 4))) (app (id 'mk-rec) (fun 'fac (fun 'n (ifexp (eqop (id 'n) (num 0)) (num 1) (mul (id 'n) (app (id 'fac) (sub (id 'n) (num 1)))))))))) (fun 'body-proc (app (fun 'fX (app (id 'fX) (id 'fX))) (fun 'fY (app (fun 'f (app (id 'body-proc) (id 'f))) (fun 'x (app (app (id 'fY) (id 'fY)) (id 'x))))))))

ex2. "{with {fib {fun {n} {if {or {= n 0} {= n 1}} 1 {+ {fib {- n 1}} {fib {- n 2}}}}}} {fib 10}}"

Desugared into\
{with {mk-rec {fun {body-proc} {with {fX {fun {fY} {with {f {fun {x} {{fY fY} x}}} {body-proc f}}}} {fX fX}}}} {with {fib {mk-rec {fun {fib} {fun {n} {if {or {= n 0} {= n 1}} 1 {+ {fib {- n 1}} {fib {- n 2}}}}}}}} {fib 10}}}

Parsing Result\
(app (fun 'mk-rec (app (fun 'fib (app (id 'fib) (num 10))) (app (id 'mk-rec) (fun 'fib (fun 'n (ifexp (orop (eqop (id 'n) (num 0)) (eqop (id 'n) (num 1))) (num 1) (add (app (id 'fib) (sub (id 'n) (num 1))) (app (id 'fib) (sub (id 'n) (num 2)))))))))) (fun 'body-proc (app (fun 'fX (app (id 'fX) (id 'fX))) (fun 'fY (app (fun 'f (app (id 'body-proc) (id 'f))) (fun 'x (app (app (id 'fY) (id 'fY)) (id 'x))))))))

ex3. "{with {count {fun {n} {if {= n 0} 0 {+ 1 {count {- n 1}}}}}} {count 8}}"

Desugared into\
{with {mk-rec {fun {body-proc} {with {fX {fun {fY} {with {f {fun {x} {{fY fY} x}}} {body-proc f}}}} {fX fX}}}} {with {count {mk-rec {fun {count} {fun {n} {if {= n 0} 0 {+ 1 {count {- n 1}}}}}}}} {count 8}}}

Parsing Result\
(app (fun 'mk-rec (app (fun 'count (app (id 'count) (num 8))) (app (id 'mk-rec) (fun 'count (fun 'n (ifexp (eqop (id 'n) (num 0)) (num 0) (add (num 1) (app (id 'count) (sub (id 'n) (num 1)))))))))) (fun 'body-proc (app (fun 'fX (app (id 'fX) (id 'fX))) (fun 'fY (app (fun 'f (app (id 'body-proc) (id 'f))) (fun 'x (app (app (id 'fY) (id 'fY)) (id 'x))))))))


**not recursion examples**

ex1. {with {x 3} {with {f {fun {y} {+ x y}}} {with {x 5} {f 4}}}}
Parsing Result\
(app (fun 'x (app (fun 'f (app (fun 'x (app (id 'f) (num 4))) (num 5))) (fun 'y (add (id 'x) (id 'y))))) (num 3))

Interpreting Result\
(numV 7)

ex2. {with {z {fun {x} {+ x y}}} {with {y 10} z}}
Parsing Result\
(app (fun 'z (app (fun 'y (id 'z)) (num 10))) (fun 'x (add (id 'x) (id 'y))))

Interpreting Result\
(closureV 'x (add (id 'x) (id 'y)) (mtSub))
