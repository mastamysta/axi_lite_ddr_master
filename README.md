# axi_lite_ddr_master
 An AXI-lite master to read & write DDR3 via the Xilix 7 Series MIG (without using the seemingly obligatory soft-processor).

 I was a bit miffed at the lack of documentation around getting the Xilinx MIG IP core to work without a MicroBlaze or some other soft-processor, so I did this project to try and find a nice way of writing to the DDR3 memory on my Arty board.

 The main 'gotchas' I discovered during this project are:
  - The MIG IP core expects contain the fastest internal clock signal on the entire board, so all other modules in the design should run off the ui_clk signal it divides for you.
  - Implementing an AXI-lite master rather than AXI4 full limited the maximum write speed significantly (More details on that to follow).
  - The MIG IP core can take *hundreds of miliseconds* to initialise! It is a very good idea to wait on the init_calib_complete signal after a reset.
  - AXI SmartConnect cannot map addresses if you don't supply a base address in the Address Editor (duh).

# Read/Write Speed Discussion
 The state machine I've left in the project writes & reads (interleaved) 100MB of data from memory before entering an idle state. On my Artix-7 L1 device, that takes roughly 13 seconds. Skipping the reads completely, this process takes only 4 seconds.

 Data rates:
  Read - 88Mbps
  Write - 200Mbps

 There are a couple of easy ways to get closer to the 667Mbps maximum data rate acheivable with the DDR3 chip and the L1 speed rating:
  - The AXI-lite master state machine spends 1/4 of its cycles in the NEXT_ITER state, so there would be a ~25% speed improvement from optimising that state out and moving its logic into other blocks.
  - Upgrading the master to AXI4 full would enable burst-read/writes. This would massively improve speed because burst operation spend far fewer cycles handshaking than consecutive single-address operations.
  - Moving to AXI4 full would also allow the SmartConnect to be removed, which *may* give a performance improvement but would remove the possibility of having multiple masters over the DDR3 RAM.
