test: testbench_0.ok testbench_1.ok

synth: synth.out
	cat synth.out

synth.out: rvb_full.v synth.py
	python3 synth.py

run: system firmware.hex
	vvp -N ./system

firmware.hex: start.S firmware.c hex8tohex32.py
	/opt/riscv64b/bin/riscv64-unknown-elf-gcc -march=rv32imb -mabi=ilp32 -o firmware.elf -Wall -Os -ffreestanding -nostdlib \
			-Wl,-Bstatic,-T,sections.lds,-Map,firmware.map,--strip-debug start.S firmware.c
	/opt/riscv64b/bin/riscv64-unknown-elf-objcopy -O verilog firmware.elf firmware.tmp
	python3 hex8tohex32.py firmware.tmp > firmware.hex
	rm -f firmware.tmp

system: system.v picorv32.v rvb_full.v rvb_pcpi.v
	iverilog -o system system.v picorv32.v rvb_pcpi.v rvb_full.v \
			../rvb_bextdep/rvb_bextdep.v ../rvb_bitcnt/rvb_bitcnt.v ../rvb_bmatxor/rvb_bmatxor.v \
			../rvb_clmul/rvb_clmul.v ../rvb_shifter/rvb_shifter.v ../rvb_simple/rvb_simple.v

testdata: testdata.cc
	c++ -Wall -o testdata testdata.cc

testdata_0.hex: testdata
	./testdata

testdata_1.hex: testdata_0.hex

testbench_0: testbench.v rvb_full.v
	iverilog -o testbench_0 -s testbench -DTESTDATA='"testdata_0.hex"' -DENABLE_64BIT testbench.v rvb_full.v \
			../rvb_bextdep/rvb_bextdep.v ../rvb_bitcnt/rvb_bitcnt.v ../rvb_bmatxor/rvb_bmatxor.v \
			../rvb_clmul/rvb_clmul.v ../rvb_shifter/rvb_shifter.v ../rvb_simple/rvb_simple.v

testbench_1: testbench.v rvb_full.v
	iverilog -o testbench_1 -s testbench -DTESTDATA='"testdata_1.hex"' testbench.v rvb_full.v \
			../rvb_bextdep/rvb_bextdep.v ../rvb_bitcnt/rvb_bitcnt.v ../rvb_bmatxor/rvb_bmatxor.v \
			../rvb_clmul/rvb_clmul.v ../rvb_shifter/rvb_shifter.v ../rvb_simple/rvb_simple.v

testbench_0.ok: testbench_0 testdata_0.hex
	vvp -N ./testbench_0
	touch testbench_0.ok

testbench_1.ok: testbench_1 testdata_1.hex
	vvp -N ./testbench_1
	touch testbench_1.ok

clean:
	rm -f testbench.vcd testdata testdata_?.hex testbench_? testbench_?.ok synth.out synth.tmp synth.ys
	rm -f system firmware.hex firmware.elf firmware.map system.vcd

.PHONY: test synth clean
