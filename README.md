# Gnu Radio Tutorial
###### Procedure for  making OOT (Out of Tree) module - Python

#### Tutorial 1 : Multiply Block (Single Input/Output port block that takes in a value from user and multiplies with incoming signal)

Setting up a new block
```
$ gr_modtool newmod tutorial
Creating out-of-tree module in ./gr-tutorial... Done.
Use 'gr_modtool add' to add a new block to this currently empty module.
```
A new folder will be created 'gr-tutorial'
```
$ gr-tutorial$ ls
apps  cmake  CMakeLists.txt  docs  examples  grc  include  lib  Python  swig
```
We will create a block in Python that will do Multiplication
```
$ gr-tutorial$ gr_modtool add -t sync -l python
GNU Radio module name identified: tutorial
Language: Python
Enter name of block/code (without module name prefix): multiply_py_ff
Block/code identifier: multiply_py_ff
Enter valid argument list, including default arguments: multiple
Add Python QA code? [Y/n] y
Adding file 'Python/multiply_py_ff.py'...
Adding file 'Python/qa_multiply_py_ff.py'...
Editing Python/CMakeLists.txt...
Adding file 'grc/tutorial_multiply_py_ff.xml'...
Editing grc/CMakeLists.txt...
```        
3 new files are created :

```multiply_py_ff.py``` Is the functional part ```qa_multiply_py_ff.py``` Used to test the code  ```tutorial_multiply_py_ff.xml``` Used to link the block to the GRC

Open **multiply_py_ff.py** in text editor & Modify
```Python
import numpy
from gnuradio import gr 

class multiply_py_ff(gr.sync_block): 
    """
    docstring for block multiply_py_ff
    """
    def __init__(self, multiple):
        self.multiple = multiple # <---- New line added
        gr.sync_block.__init__(self,
            name="multiply_py_ff",
            in_sig=[<+numpy.float+>], # <---- Change to in_sig=[numpy.float32]
            out_sig=[<+numpy.float+>]) # <---- Change to out_sig=[numpy.float32]
        self.multiple = multiple

    #The work function is where the actual processing happens
    def work(self, input_items, output_items):
        in0 = input_items[0] 
        out = output_items[0]
        out[:] = in0 # <----Change to out[:] = in0*self.multiple
        return len(output_items[0]) 
```

Open **qa_multiply_py_ff.py** in text editor & Modify

```Python
from gnuradio import gr, gr_unittest
from gnuradio import blocks
from multiply_py_ff import multiply_py_ff

class qa_multiply_py_ff (gr_unittest.TestCase):

    def setUp (self):
        self.tb = gr.top_block ()

    def tearDown (self):
        self.tb = None

    def test_001_t (self):
    	src_data = (0, 1, -2, 5.5, -0.5) # <---- New line added
    	expected_result = (0, 2, -4, 11, -1) #Multiplying by 2 <---- New line added

    	src = blocks.vector_source_f(src_data) # <---- New line added
    	mult = multiply_py_ff(2) # <---- New line added
    	snk = blocks.vector_sink_f() # <---- New line added

    	self.tb.connect(src,mult) # <---- New line added
    	self.tb.connect(mult,snk) # <---- New line added

        self.tb.run ()
        result_data = snk.data() # <---- New line added

        self.assertFloatTuplesAlmostEqual (expected_result, result_data, 6) # <---- New line added

if __name__ == '__main__':
    gr_unittest.run(qa_multiply_py_ff, "qa_multiply_py_ff.xml")
```

Go to the /python directory and run
```
$gr-tutorial/python$ python qa_multiply_py_ff.py
.
----------------------------------------------------------------------
Ran 1 test in 0.004s

OK
```
Finally open the file **tutorial_multiply_py_ff.xml** and edit the parameters

```Python
  <param>
    <name>Multiple</name>
    <key>multiple</key>
    <type>float</type>
  </param>
  
  <sink>
    <name>in</name>
    <type>float</type>
  </sink>
  
  <source>
    <name>out</name>
    <type>float</type>
  </source>  
```

Installing the block
```
cd gr-tutorial/grc/
mkdir build
cmake ../
make
sudo make install
sudo ldconfig
```
Open GnuRadio Software, we will see the block **multiply_py_ff** is ready to use

![capture](https://user-images.githubusercontent.com/22035469/47813486-db1b1300-dd10-11e8-8f64-98f8ecfb6e20.JPG)

Draw a flow graph to test this new block

![capture](https://user-images.githubusercontent.com/22035469/47813709-77451a00-dd11-11e8-8e27-eb19788ead98.JPG)

In the GUI plot we can see the amplitude of the sine wave = 2 is being multiplied by our multiplication constant 2 and the output amplitude is 4
![capture](https://user-images.githubusercontent.com/22035469/47813782-980d6f80-dd11-11e8-9d96-8ed588d33a56.JPG)




#### Tutorial 2 : Adder Block (Two Input-One Output port block that takes in a value from user/vector source in both of the input ports and add them together)

Same procedure is followed from previous example, just the coding is different

qa_Adder.py
```Python
from gnuradio import gr, gr_unittest
from gnuradio import blocks
from Adder import Adder

class qa_Adder (gr_unittest.TestCase):

    def setUp (self):
        self.tb = gr.top_block ()

    def tearDown (self):
        self.tb = None

    def test_001_t (self):
    	src0 = blocks.vector_source_f(1, 3, 5, 7, 9)
        src1 = blocks.vector_source_f(0, 2, 4, 6, 8)
        add = Adder()
        sink = blocks.vector_sink_f()
        self.tb.connect((src0, 0), (add, 0))
        self.tb.connect((src1, 0), (add, 1))
        self.tb.connect(add, sink)
        self.tb.run ()
        self.assertEqual(sink.data(), (1, 5, 9, 13, 17))

if __name__ == '__main__':
    gr_unittest.run(qa_Adder, "qa_Adder.xml")
```

Adder.py
```Python
import numpy
from gnuradio import gr

class Adder(gr.sync_block):
    def __init__(self):
        gr.sync_block.__init__(self,
            name="Adder",
            in_sig=[numpy.float32,numpy.float32],
            out_sig=[numpy.float32])


    def work(self, input_items, output_items):
        output_items[0][:] = input_items[0] + input_items[1]
        return len(output_items[0])
```        

Adder.xml
```Python
<?xml version="1.0"?>
<block>
  <name>Adder</name>
  <key>personal_Adder</key>
  <category>[personal]</category>
  <import>import personal</import>
  <make>personal.Adder()</make>

  <sink>
    <name>in0</name>
    <type>float</type>
  </sink>

  <sink>
    <name>in1</name>
    <type>float</type>
  </sink>

  <source>
    <name>out</name>
    <type>float</type>
  </source>
</block>
```

![capture](https://user-images.githubusercontent.com/22035469/47840874-56b2a980-dd7d-11e8-83f8-069559053a68.JPG)

![capture](https://user-images.githubusercontent.com/22035469/47840922-7d70e000-dd7d-11e8-9ed3-156645a105f5.JPG)
