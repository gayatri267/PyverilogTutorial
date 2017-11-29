Pyverilog
==============================

Python-based Hardware Design Processing Toolkit for Verilog HDL
Pyverilog is an open-source hardware design processing toolkit for Verilog HDL.

This software includes various tools for Verilog HDL design.
* vparser: Code parser to generate AST (Abstract Syntax Tree) from source codes of Verilog HDL.
* dataflow: Dataflow analyzer with an optimizer to remove redundant expressions and some dataflow handling tools.
* controlflow: Control-flow analyzer with condition analyzer that identify when a signal is activated.
* ast\_code\_generator: Verilog HDL code generator from AST(Abstract Syntax Tree).

You can create your own design analyzer, code translator and code generator of Verilog HDL based on this toolkit.

Prerequisites
==============================

* Python (2.7, 3.3 or later)
* Icarus Verilog (0.9.6 or later)
   - pyverilog.vparser.preprocessor.py uses 'iverilog -E' command as the preprocessor.
* Graphviz and Pygraphviz (Python3 does not support Pygraphviz)
   - pyverilog.dataflow.graphgen and pyverilog.controlflow.controlflow (without --nograph option) use Pygraphviz (on Python 2.7).
* Jinja2 (2.7 or later)
   - ast\_code\_generator requires jinja2 module.

Hands on Tutorial
==============================

Git Clone Our Tutorial
------------------------------

    git clone git@github.com:gayatri267/PyverilogTutorial.git
    

Do the following Installations
------------------------------------
Please make sure you have python2.7 Installed before continuing with this tutorial.
Some of the commands in this tutorial can only be run using Python 2.7, while others can be run using Python 2.7 or Python 3
(We will specify during the commands if python2.7 is to be explicitly used)

1. Go into the pyverilog-0.9.1 directory in the cloned repository
    cd pyverilog-0.9.1
    
2. Install the pyverilog library using setup.py.
    * If Python 2.7 is used,
    ```
    python setup.py install
    ```

    * If Python 3.x is used,
    ```
    python3 setup.py install
    ```
    
3. Install iverilog as below 
    ```
    sudo apt-get install iverilog
    ```
    
4. Install Pygraphviz as below (Note that Python3 does not support Pygraphviz and hence only Python2.7 is to be installed)
    ```
    sudo apt-get install -y python-pygraphviz
    ```
    
5. Install Jinja 2 as below
    ```
    sudo apt install python-pip
    pip install jinja2
    ```
    
Example 1 (test.v)
-----------------------------
Let's try to use pyverilog tools on the verilog module test.v(already present in the pyverilog-0.9.1 directory)
This sample design adds the input value internally whtn the enable signal is asserted. Then it outputs its partial value to the LED.

###Code parser###
Code parer is the syntax analysis. Please type the command as below.

    python pyverilog/vparser/parser.py test.v

The result of syntax analysis is displayed.

```
Source: 
  Description: 
    ModuleDef: top
      Paramlist: 
      Portlist: 
        Ioport: 
          Input: CLK, False
            Width: 
              IntConst: 0
              IntConst: 0
        Ioport: 
          Input: RST, False
            Width: 
              IntConst: 0
              IntConst: 0
        Ioport: 
          Input: enable, False
            Width: 
              IntConst: 0
              IntConst: 0
        Ioport: 
          Input: value, False
            Width: 
              IntConst: 31
              IntConst: 0
        Ioport: 
          Output: led, False
            Width: 
              IntConst: 7
              IntConst: 0
      Decl: 
        Reg: count, False
          Width: 
            IntConst: 31
            IntConst: 0
      Decl: 
        Reg: state, False
          Width: 
            IntConst: 7
            IntConst: 0
      Assign: 
        Lvalue: 
          Identifier: led
        Rvalue: 
          Partselect: 
            Identifier: count
            IntConst: 23
            IntConst: 16
      Always: 
        SensList: 
          Sens: posedge
            Identifier: CLK
        Block: None
          IfStatement: 
            Identifier: RST
            Block: None
              NonblockingSubstitution: 
                Lvalue: 
                  Identifier: count
                Rvalue: 
                  IntConst: 0
              NonblockingSubstitution: 
                Lvalue: 
                  Identifier: state
                Rvalue: 
                  IntConst: 0
            Block: None
              IfStatement: 
                Eq: 
                  Identifier: state
                  IntConst: 0
                Block: None
                  IfStatement: 
                    Identifier: enable
                    NonblockingSubstitution: 
                      Lvalue: 
                        Identifier: state
                      Rvalue: 
                        IntConst: 1
                IfStatement: 
                  Eq: 
                    Identifier: state
                    IntConst: 1
                  Block: None
                    NonblockingSubstitution: 
                      Lvalue: 
                        Identifier: state
                      Rvalue: 
                        IntConst: 2
                  IfStatement: 
                    Eq: 
                      Identifier: state
                      IntConst: 2
                    Block: None
                      NonblockingSubstitution: 
                        Lvalue: 
                          Identifier: count
                        Rvalue: 
                          Plus: 
                            Identifier: count
                            Identifier: value
                      NonblockingSubstitution: 
                        Lvalue: 
                          Identifier: state
                        Rvalue: 
                          IntConst: 0
```

###Dataflow analyzer###
Let's try dataflow analysis. It is used to establish the relationship between outputs with inputs and states

    python3 pyverilog/dataflow/dataflow_analyzer.py -t top test.v 

The result of each signal definition and each signal assignment are displayed.

```
Directive:
Instance:
(top, 'top')
Term:
(Term name:top.led type:{'Output'} msb:(IntConst 7) lsb:(IntConst 0))
(Term name:top.enable type:{'Input'} msb:(IntConst 0) lsb:(IntConst 0))
(Term name:top.CLK type:{'Input'} msb:(IntConst 0) lsb:(IntConst 0))
(Term name:top.count type:{'Reg'} msb:(IntConst 31) lsb:(IntConst 0))
(Term name:top.state type:{'Reg'} msb:(IntConst 7) lsb:(IntConst 0))
(Term name:top.RST type:{'Input'} msb:(IntConst 0) lsb:(IntConst 0))
(Term name:top.value type:{'Input'} msb:(IntConst 31) lsb:(IntConst 0))
Bind:
(Bind dest:top.count tree:(Branch Cond:(Terminal top.RST) True:(IntConst 0) False:(Branch Cond:(Operator Eq Next:(Terminal top.state),(IntConst 0)) False:(Branch Cond:(Operator Eq Next:(Terminal top.state),(IntConst 1)) False:(Branch Cond:(Operator Eq Next:(Terminal top.state),(IntConst 2)) True:(Operator Plus Next:(Terminal top.count),(Terminal top.value)))))))
(Bind dest:top.state tree:(Branch Cond:(Terminal top.RST) True:(IntConst 0) False:(Branch Cond:(Operator Eq Next:(Terminal top.state),(IntConst 0)) True:(Branch Cond:(Terminal top.enable) True:(IntConst 1)) False:(Branch Cond:(Operator Eq Next:(Terminal top.state),(IntConst 1)) True:(IntConst 2) False:(Branch Cond:(Operator Eq Next:(Terminal top.state),(IntConst 2)) True:(IntConst 0))))))
(Bind dest:top.led tree:(Partselect Var:(Terminal top.count) MSB:(IntConst 23) LSB:(IntConst 16)))
```

To view the result of dataflow analysis as a picture file, need to run the command as below (we select output port 'led' as the target for example)

    python3 pyverilog/dataflow/graphgen.py -t top -s top.led test.v 

out.png file will now be generated which has the definition of 'led' is a part-selection of 'count' from 23-bit to 16-bit.

![out.png](http://cdn-ak.f.st-hatena.com/images/fotolife/s/sxhxtxa/20140101/20140101045641.png)

###Control-flow analyzer###
Control-flow analysis can be used to picturize how the state diagram of the RTL module look like

    python2.7 pyverilog/controlflow/controlflow_analyzer.py -t top test.v 
(Note that only python2.7 can be used to this command as it internally uses Pygraphviz)

We get the output as below, which shows the state machine structure and transition conditions to the next state in the state machine.

```
FSM signal: top.count, Condition list length: 4
FSM signal: top.state, Condition list length: 5
Condition: (Ulnot, Eq), Inferring transition condition
Condition: (Eq, top.enable), Inferring transition condition
Condition: (Ulnot, Ulnot, Eq), Inferring transition condition
# SIGNAL NAME: top.state
# DELAY CNT: 0
0 --(top_enable>'d0)--> 1
1 --None--> 2
2 --None--> 0
Loop
(0, 1, 2)
```

top_state.png is also generated which is the graphical representation of the state machine.

![top_state.png](http://cdn-ak.f.st-hatena.com/images/fotolife/s/sxhxtxa/20140101/20140101045835.png)

###Code Generator###
Finally, pyverilog can be used to generate verilog code from the AST representation of RTL. We will be using 'test.py' for demonstrate
A Verilog HDL code is represented by using the AST classes defined in 'vparser.ast'.
Run the below command to see how AST representation in test.py gets translated to verilog code
```
python test.py
```

Verilog code generated from the AST instances is as below:

```verilog

module top
 (
  input [0:0] CLK, 
input [0:0] RST, 
output [7:0] led

 );
  assign led = 8;
endmodule

```