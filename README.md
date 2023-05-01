Download Link: https://assignmentchef.com/product/solved-cda4102_cda5155-project-simulator-for-a-pipelined-processor
<br>
In this project you will create a simulator for a pipelined processor. Your simulator should be capable of loading a specified MIPS text file (see sample.txt) and generate the cycle-by-cycle simulation of the MIPS code. It should also produce/print the contents of registers, queues, and memory data for each cycle. <strong>No need to handle exceptions of any kind. </strong>For example, test cases will not try to execute data (from data segment) as instructions, or load/store data from instruction segment. Similarly, there will not be any invalid opcodes or less than 32-bit instructions in the input file, etc.




Please develop your project <strong>in one source file</strong> (written in <strong>C</strong>, <strong>C++</strong>, <strong>Java</strong> or <strong>Python</strong>) to avoid the stress of combining multiple files before submission and making sure it still works correctly. Please follow the <strong>Submission Policy</strong> which is at the end of this document to submit your source file. Your MIPS simulator (with executable name as <strong>MIPSsim</strong>) should accept an input file (inputfilename.txt) in the following command format and produce output file (simulation.txt) that contains the simulation trace. In this project, you do not have to produce disassembly file.

MIPSsim inputfilename.txt

Correct handling of the sample input file (with possible different data values) will be used to determine 60% of the credit. The remaining 40% will be determined from other test cases that you will not have access prior to grading. It is recommended that you construct your own sample input files with which to further test your simulator.

<strong>Instruction Format: </strong>The instruction format and other details (e.g., starting address) remain the same as <strong>Project 1</strong> (see the PDF: <a href="https://www.cise.ufl.edu/~prabhat/Teaching/cda5155-f20/projects/project1.html">https://www.cise.ufl.edu/~prabhat/Teaching/cda5155</a><a href="https://www.cise.ufl.edu/~prabhat/Teaching/cda5155-f20/projects/project1.html">–</a><a href="https://www.cise.ufl.edu/~prabhat/Teaching/cda5155-f20/projects/project1.html">f20/projects/project1.html</a><a href="https://www.cise.ufl.edu/~prabhat/Teaching/cda5155-f20/projects/project1.html">)</a>.

<strong>Pipeline Description:</strong>

The entire pipeline is synchronized by a single clock signal. The white boxes represent the functional units, the blue boxes represent queues between the units, the yellow boxes represent registers and the green one is the memory unit.  In the remainder of this section, we describe the functionality of each of the units/queues/memories in detail. We use the terms “the end of cycle” and “the beginning of cycle” interchangeably in following discussion. Both of them refer to the rising edge of the clock signal, i.e., the end of the previous cycle implies the beginning of the next cycle.

<strong>Instruction Fetch/Decode (IF):  </strong>

Instruction Fetch/Decode unit can <strong>fetch and decode</strong> at most <strong>two</strong> instruction at each cycle (in program order).  The unit should check all the following conditions before it can fetch further instructions.

<ul>

 <li>If the fetch unit is stalled at the end of last cycle, no instruction can be fetched at the current cycle. The fetch unit can be stalled due to a branch instruction.</li>

 <li>If there is no empty slot in the Pre-issue queue at the end of the last cycle, no instruction can be fetched at the current cycle.</li>

</ul>

Normally, the whole fetch-decode operation can be finished in 1 cycle. The decoded instruction will be placed in Pre-issue queue before the end of the current cycle. If a branch instruction is fetched, the fetch unit will try to read all the necessary registers to calculate the target address. If all the registers are ready (or target is immediate), it will update PC before the end of the current cycle. Otherwise the unit is stalled until the required registers are available. In other words, if registers are ready (or immediate target value) at the end of the last cycle, the branch does not introduce any penalty.

There are two possible scenarios when a branch instruction (J, JR, BEQ, BLTZ, BGTZ) is fetched along with another instruction. The branch can be the first instruction or the last instruction in the pair (remember, up to two instructions can be fetched per cycle). When a branch instruction is fetched with its next (in-order) instruction (first scenario), the next instruction will be discarded immediately (needs to be re-fetched again based on the branch outcome). When the branch is the last instruction in the pair (second scenario), both are decoded as usual.

<strong> </strong>

Note that the register accesses are synchronized. The value read from register file in the current cycle is the value of corresponding register at the end of the previous cycle. In other words, any functional units cannot obtain the new register values written by WB in the same cycle.




When a BREAK instruction is fetched, the fetch unit will not fetch any more instructions.

All branch instructions, BREAK instruction and NOP instruction will not be written to Pre-issue queue. It is important to note that we still need free entries in the pre-issue queue at the end of last cycle before the fetch unit fetches them, because the fetch cannot predict the types of instructions before fetching and decoding them.

<strong>Pre-issue Queue: </strong>Pre-Issue Queue has 4 entries; each entry can store one instruction. The instructions are sorted by their program order, the entry 0 always contains the oldest and the entry 3 contains the newest.

<strong>Issue Unit: </strong>Issue unit follows the basic Scoreboard algorithm to read operands from Register File and issue instructions when all the source operands are ready. It can issue at most <strong>two</strong> instruction <strong>out-oforder</strong> per cycle. It can send at most <strong>one</strong> load or store (LW or SW) instructions per cycle to the Pre-ALU1 queue, and at most <strong>one</strong> non load/store (except LW or SW) instructions per cycle to the Pre-ALU2 queue. When an instruction is issued, it is removed from the Pre-issue Queue before the end of current cycle. The issue unit searches from entry 0 to entry 3 (in that order) of Pre-issue queue and issues instructions if: <strong> </strong>

<ul>

 <li>No structural hazards (the corresponding queue, i.e., Pre-ALU1 or Pre-ALU2 has empty slots at the end of the last cycle);</li>

 <li>No RAW or WAW hazards with active instructions (issued but not finished, or earlier not-issued instructions).</li>

 <li>If two instructions are issued in a cycle, you need to make sure that there are no WAW or WAR hazards between them.</li>

 <li>No WAR hazards with earlier not-issued instructions;</li>

 <li>For MEM instructions, all the source registers are ready at the end of the last cycle.</li>

 <li>The load instruction must wait until all the previous stores are issued.</li>

 <li>The stores must be issued in order.</li>

</ul>

<strong>Pre-ALU1 queue: </strong>The Pre-ALU1 queue has two entries. Each entry can store one memory (LW or SW) instruction with its operands. The queue is managed as <strong>FIFO</strong> (in-order) queue.

<strong>Pre-ALU2 queue: </strong>The Pre-ALU2 queue has two entries. Each entry can store one ALU (any instruction except LW or SW) instruction with its operands. The queue is managed as <strong>FIFO</strong> (in-order) queue.

<strong>ALU1: </strong>ALU1 handles the calculation of address for memory (LW and SW) instructions. ALU1 can fetch one instruction each cycle from the Pre-ALU1 queue, removes it from the Pre-ALU1 queue (at the beginning of the current cycle) and computes it.The instruction and its result will be written into the PreMEM queue at the end of the current cycle. Note that ALU1 starts execution even if the Pre-MEM queue is occupied (full) at the beginning of the current cycle. This is because MEM is guaranteed to consume (remove) the entry from the Pre-MEM queue before the end of the current cycle.

<strong>ALU2: </strong>ALU2 handles the calculation of all non-memory instructions. All the instructions take one cycle.  The ALU can fetch one instruction each cycle from the Pre-ALU2 queue, removes it from the Pre-ALU2 queue (at the beginning of the current cycle) and compute it.The instruction and its result will be written into the Post-ALU2 queue at the end of the current cycle. Note that ALU2 starts execution even if the Post-ALU2 queue is occupied (full) at the beginning of the current cycle. This is because WB is guaranteed to consume (remove) the entry from the Post-ALU2 queue before the end of the current cycle.

<strong>Post-ALU2 queue: </strong>This queue has one entry. This entry can store one instruction with destination register id and the result.

<strong>Pre-MEM queue: </strong>The Pre-MEM queue has one entry. This entry can store one memory instruction (LW, SW) with its operands. <strong> </strong>

<strong>MEM Unit: </strong>The MEM unit handles LW and SW instructions. It reads from Pre-MEM queue. For LW instruction, MEM takes one cycle to read the data from memory. When a LW instruction finishes, the instruction with destination register id and the data will be written to the Post-MEM queue before the end of the current cycle. Note that MEM starts execution even if the Post-MEM queue is occupied (full) at the beginning of the current cycle. This is because WB is guaranteed to consume (remove) the entry from the Post-MEM queue before the end of the current cycle.For SW instruction, MEM also takes one cycle to finish (write the data to memory). When a SW instruction finishes, nothing would be sent to Post-MEM queue.

<strong>Post-MEM queue: </strong>Post-MEM queue has one entry that can store one LW instruction with destination register id and data. <strong> </strong>

<strong>WB Unit: </strong>WB unit can execute up to <strong>two</strong> writebacks in one cycle consisting of at most one from PostMEM queue and at most one from Post-ALU2 queue. It updates the Register File based on the content of both Post-ALU2 Queue (any instruction except LW or SW) and Post-MEM Queue (LW). The update finishes before the end of the cycle. The new value will be available at the beginning of the next cycle.

<strong>PC: </strong>It records the address of the next instruction to fetch. It should be set to 256 at the initialization. <strong> </strong>

<strong>Register File: </strong>There are 32 registers. Assume that there are sufficient read/write ports to support all kinds of read write operations from different functional units.

<strong>Notes on Pipelines: </strong>

<ol>

 <li>The simulation finishes when the BREAK instruction is fetched. In other words, the last clock cycle that you print in the simulation output is the one where BREAK is fetched (shown in the “Executed Instruction” field). In other words, there may be unfinished instructions in the pipeline when you stop the simulation (see the sample_simulation.txt to see it ended).</li>

 <li>No data forwarding.</li>

 <li>No delay slot will be used for branch instructions.</li>

 <li>Different instructions take different stages to finish.

  <ol>

   <li>NOP, Branch, BREAK: only IF;</li>

   <li>SW: IF, Issue, ALU1, MEM;</li>

   <li>LW: IF, Issue, ALU1, MEM, WB;</li>

   <li>Other instructions: IF, Issue, ALU2, WB.</li>

  </ol></li>

</ol>

<strong> </strong>

<strong> </strong>

<strong> </strong>

<strong> </strong>

<strong> </strong>

<strong>Output format </strong>

For each cycle, you should output the whole state of the processor and the memory <strong>at the end of each cycle</strong>. If any entry in a queue is empty, no content for that entry should be printed. The instruction should be printed as in Project 1.

20 hyphens and a new line Cycle  [value]: &lt;blank_line&gt; IF Unit:

&lt;tab&gt;Waiting Instruction: [instruction waiting for its operand] &lt;tab&gt;Executed Instruction: [instruction executed in this cycle] Pre-Issue Queue:

&lt;tab&gt;Entry 0: [instruction]

&lt;tab&gt;Entry 1: [instruction]

&lt;tab&gt;Entry 2: [instruction]

&lt;tab&gt;Entry 3: [instruction] Pre-ALU1 Queue:

&lt;tab&gt;Entry 0: [instruction]

&lt;tab&gt;Entry 1: [instruction]

Pre-MEM Queue: [instruction] Post-MEM Queue: [instruction] Pre-ALU2 Queue:

&lt;tab&gt;Entry 0: [instruction]

&lt;tab&gt;Entry 1: [instruction]

Post-ALU2 Queue: [instruction]

&lt; blank_line &gt;

Registers

R00:&lt; tab &gt;&lt; int(R0) &gt;&lt; tab &gt;&lt; int(R1) &gt;..&lt; tab &gt;&lt; int(R7) &gt;

R08:&lt; tab &gt;&lt; int(R8) &gt;&lt; tab &gt;&lt; int(R9) &gt;..&lt; tab &gt;&lt; int(R15) &gt;

R16:&lt; tab &gt;&lt; int(R16) &gt;&lt; tab &gt;&lt; int(R17) &gt;..&lt; tab &gt;&lt; int(R23) &gt;

R24:&lt; tab &gt;&lt; int(R24) &gt;&lt; tab &gt;&lt; int(R25) &gt;..&lt; tab &gt;&lt; int(R31) &gt;

&lt;blank line&gt;

Data

&lt; firstDataAddress &gt;:&lt; tab &gt;&lt; display 8 data words as integers with tabs in between &gt;

….. &lt; continue until the last data word &gt;