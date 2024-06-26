**************************
*2A03 technical reference*
**************************
Brad Taylor (BTTDgroup@hotmail.com)
First release: April 23rd, 2004

Thanks to hundreds of selfless people around the world, a document like this 
one can exist because people who love NES/FC/FDS research & development have 
chosen to share their findings, experience, and knowledge. With 
"http://nesdev.parodius.com" and the Membled Messageboards, public-domain 
NES software/hardware/emulator development has already hit a new standard of 
excellence, and is attracting more people nowadays then ever before because 
of this.

Note: to display this document properly, your text viewer needs two things: 
1. support for the classic VGA-based text mode 256 character set with 
line-drawing characters. 2. word-wrap. windows notepad can easially do both 
if you change the font over to terminal style.


+----------------+
|Topics discussed|
+----------------+
Integrated components overview
2A03 pin nomenclature & signal descriptions
6502 opcode pattern tables
Introduction to sound channels
Low frequency programmable timer
2A03 internal hardware port map
Microarchitecture of basic sound channels


********************************
*Integrated components overview*
********************************
The 2A03 is a custom integrated circuit used as the heart of NES game 
consoles and Family Computers. To avoid costly glue logic, Nintendo squeezed 
alot of hardware (alot for the time, which was like 1982) inside this chip. 
Here is a list of known integrated components found in the 2A03 (* prefix 
indicates simple hardware discussed next).

- stock NMOS 6502 microprocessor lacking decimal mode support
- low frequency programmable timer
- two nearly-identical rectangle wave function generators
- triangle wave function generator
- random wavelength function generator
- audio sample playback unit (delta modulation channel)
- one shot programmable DMA transfer unit
* master dodecade clock divider
* two 6502 address decoders for $4016R and $4017R
* 3-bit register and address decoder for $4016W


*********************************************
*2A03 pin nomenclature & signal descriptions*
*********************************************
This chapter owes thanks to Kevin Horton for his help with alot of my early 
technical questions on the NES back in 1999, and his excellent "NES Cart 
Types" document.

          ___  ___
         |*  \/   |
ROUT  <01]        [40<  VCC
COUT  <02]        [39>  $4016W.0
/RES >03]        [38>  $4016W.1
A0   <04]        [37>  $4016W.2
A1   <05]        [36>  /$4016R
A2   <06]        [35>  /$4017R
A3   <07]        [34>  R/W
A4   <08]        [33<  /NMI
A5   <09]        [32<  /IRQ
A6   <10]  2A03  [31>  PHI2
A7   <11]        [30<  ---
A8   <12]        [29<  CLK
A9   <13]        [28]  D0
A10  <14]        [27]  D1
A11  <15]        [26]  D2
A12  <16]        [25]  D3
A13  <17]        [24]  D4
A14  <18]        [23]  D5
A15  <19]        [22]  D6
VEE  >20]        [21]  D7
         |________|


ROUT: this signal carries the mixed outputs for both internal rectangle wave 
function generators (see  "4-bit DAC" section for details).

COUT: this signal carries the combined outputs for an internal triangle 
wave/random wave function generator, and a programmable 7-bit DAC controlled 
by a delta counter/DMA timer unit combination (see  "4-bit DAC" section for 
details).

/RES: hard reset on zero. Resets the status of several internal 2A03 
registers, and the 6502.

A0-A15: the 6502's address bus output pins.

VEE, VCC: ground, and +5VDC power signals, respectfully.

D0-D7: the 6502's data bus.

CLK: this is the 2A03's master clock input line (236250/11 KHz), and clocks 
an internal divide-by-12 counter.

---: normally grounded in NES/FC consoles, this pin has unknown 
functionality. I suspect that it is an input controlling somthing, since the 
pin does draw a little current.

PHI2: this output is the divide-by-12 result of the CLK signal (1.79 MHz). 
The internal 6502 along with function generating hardware, is clocked off 
this frequency, and is available externally here so that it can be used as a 
data bus enable signal (when at logic level 1) for external 6502 address 
decoder logic. The signal has a 62.5% duty cycle.

/IRQ: interrupts the 6502 when this pin is set to zero while the 6502's 
internal interrupt mask flag is 0.

/NMI: NMI's the 6502 on a negative edge signal transition (1->0).

R/W: direction of 6502's data bus (0=write;1=read).

/$4017R: goes active (zero) when A0-A15 = $4017, R/W = 0, and PHI2 = 1. This 
informs an external 3-state inverter to throw controller port data onto the 
D0-D7 lines.

/$4016R: goes active (zero) when A0-A15 = $4016, R/W = 0, and PHI2 = 1.

$4016W.0, $4016W.1, $4016W.2: these signals represent the real-time status 
of a 3 bit writable register located at $4016 in the 6502 memory map. In 
NES/FC consoles, $4016W.0 is used as a strobe line for the CMOS 4021 shift 
register used inside NES/FC controllers.


****************************
*6502 opcode pattern tables*
****************************
Below are two tables which displays the 6502 opcode matrix, and clearly 
exposes all the wierd ways the opcode number relates to the operation of the 
instruction. This new version has John West and Marko MŠkelŠ to thank for 
their excellent "NMOS 65xx Instruction Set" documentation, available at 
http://nesdev.parodius.com/. This document is recommend literature for those 
of you out there who are looking for an exteremely detailed look at how the 
6502 works on a per-clock cycle basis (useful for correct emulation of 
instructions with dead cycles, like BRK, JSR, RTI, RTS, push/pop, implied, 
and read-modify-write ones (RMW are the most important to implement 
properly)).


+-------------+
|table 1 notes|
+-------------+
abbr.   what it means
-----   -------------
IMD     #$xx
REL     $xx,PC
0PG     $xx
0PX     $xx,X
0PY     $xx,Y
ABS     $xxxx
ABX     $xxxx,X
ABY     $xxxx,Y
IND     ($xxxx)
NDX     ($xx,X)
NDY     ($xx),Y

1ÍÍÍÍÍÍÍÍÍÑÍÍÍÍÍÍÍÑÍÍÍÍÍÍÍÑÍÍÍÍÍÍÍÑÍÍÍÍÍÍÍÑÍÍÍÍÍÍÍÑÍÍÍÍÍÍÍÑÍÍÍÍÍÍÍÑÍÍÍÍÍÍÍ»
º7654 3210³xx0 00x³xx1 00x³xx0 10x³xx1 10x³xx0 01x³xx1 01x³xx0 11x³xx1 11xº
ÇÄÄÄÄÄÄÄÄÄÅÄÄÄÄÄÄÄÅÄÄÄÄÄÄÄÅÄÄÄÄÄÄÄÅÄÄÄÄÄÄÄÅÄÄÄÄÄÄÄÅÄÄÄÄÄÄÄÅÄÄÄÄÄÄÄÅÄÄÄÄÄÄÄ¶
º000x xx00³BRK IMD³BPL    ³PHP    ³CLC    ³nop opg³nop opx³nop abs³nop abxº
º001x xx00³JSR ABS³BMI    ³PLP    ³SEC    ³BIT 0PG³nop opx³BIT ABS³nop abxº
º010x xx00³RTI    ³BVC    ³PHA    ³CLI    ³nop opg³nop opx³JMP ABS³nop abxº
º011x xx00³RTS    ³BVS    ³PLA    ³SEI    ³nop opg³nop opx³JMP IND³nop abxº
º100x xx00³nop imd³BCC    ³DEY    ³TYA    ³STY 0PG³STY 0PX³STY ABS³shy abxº
º101x xx00³LDY IMD³BCS    ³TAY    ³CLV    ³LDY 0PG³LDY 0PX³LDY ABS³LDY ABXº
º110x xx00³CPY IMD³BNE    ³INY    ³CLD    ³CPY 0PG³nop opx³CPY ABS³nop abxº
º111x xx00³CPX IMD³BEQ    ³INX    ³SED    ³CPX 0PG³nop opx³CPX ABS³nop abxº
º000x xx10³       ³       ³ASL A  ³nop    ³ASL 0PG³ASL 0PX³ASL ABS³ASL ABXº
º001x xx10³       ³       ³ROL A  ³nop    ³ROL 0PG³ROL 0PX³ROL ABS³ROL ABXº
º010x xx10³       ³       ³LSR A  ³nop    ³LSR 0PG³LSR 0PX³LSR ABS³LSR ABXº
º011x xx10³       ³       ³ROR A  ³nop    ³ROR 0PG³ROR 0PX³ROR ABS³ROR ABXº
º100x xx10³nop imd³       ³TXA    ³TXS    ³STX 0PG³STX 0PY³STX ABS³shx abyº
º101x xx10³LDX IMD³       ³TAX    ³TSX    ³LDX 0PG³LDX 0PY³LDX ABS³LDX ABYº
º110x xx10³nop imd³       ³DEX    ³nop    ³DEC 0PG³DEC 0PX³DEC ABS³DEC ABXº
º111x xx10³nop imd³       ³NOP    ³nop    ³INC 0PG³INC 0PX³INC ABS³INC ABXº
º000x xx01³ORA NDX³ORA NDY³ORA IMD³ORA ABY³ORA 0PG³ORA 0PX³ORA ABS³ORA ABXº
º001x xx01³AND NDX³AND NDY³AND IMD³AND ABY³AND 0PG³AND 0PX³AND ABS³AND ABXº
º010x xx01³EOR NDX³EOR NDY³EOR IMD³EOR ABY³EOR 0PG³EOR 0PX³EOR ABS³EOR ABXº
º011x xx01³ADC NDX³ADC NDY³ADC IMD³ADC ABY³ADC 0PG³ADC 0PX³ADC ABS³ADC ABXº
º100x xx01³STA NDX³STA NDY³nop imd³STA ABY³STA 0PG³STA 0PX³STA ABS³STA ABXº
º101x xx01³LDA NDX³LDA NDY³LDA IMD³LDA ABY³LDA 0PG³LDA 0PX³LDA ABS³LDA ABXº
º110x xx01³CMP NDX³CMP NDY³CMP IMD³CMP ABY³CMP 0PG³CMP 0PX³CMP ABS³CMP ABXº
º111x xx01³SBC NDX³SBC NDY³SBC IMD³SBC ABY³SBC 0PG³SBC 0PX³SBC ABS³SBC ABXº
º000x xx11³slo ndx³slo ndy³anc imd³slo aby³slo opg³slo opx³slo abs³slo abxº
º001x xx11³rla ndx³rla ndy³anc imd³rla aby³rla opg³rla opx³rla abs³rla abxº
º010x xx11³sre ndx³sre ndy³asr imd³sre aby³sre opg³sre opx³sre abs³sre abxº
º011x xx11³rra ndx³rra ndy³arr imd³rra aby³rra opg³rra opx³rra abs³rra abxº
º100x xx11³sax ndx³sha ndy³ane imd³shs aby³sax opg³sax opy³sax abs³sha abyº
º101x xx11³lax ndx³lax ndy³lxa imd³las aby³lax opg³lax opy³lax abs³lax abyº
º110x xx11³dcp ndx³dcp ndy³sbx imd³dcp aby³dcp opg³dcp opx³dcp abs³dcp abxº
º111x xx11³isb ndx³isb ndy³sbc imd³isb aby³isb opg³isb opx³isb abs³isb abxº
ÈÍÍÍÍÍÍÍÍÍÏÍÍÍÍÍÍÍÏÍÍÍÍÍÍÍÏÍÍÍÍÍÍÍÏÍÍÍÍÍÍÍÏÍÍÍÍÍÍÍÏÍÍÍÍÍÍÍÏÍÍÍÍÍÍÍÏÍÍÍÍÍÍÍ¼
2ÍÍÍÍÍÍÍÍÑÍÍÑÍÍÍÑÍÍÍÑÍÍÍÑÍÍÍÑÍÍÍÑÍÍÍÑÍÍÍÑÍÍÍÑÍÍÍÑÍÍÍÑÍÍÍÑÍÍÍÑÍÍÍÑÍÍÍÑÍÍÍÑÍÍÍ»
ºadr.mode³++³ 00³ 20³ 40³ 60³ 80³ A0³ C0³ E0³ 02³ 22³ 42³ 62³ 82³ A2³ C2³ 
E2º
ÇÄÄÄÄÄÄÄÄÅÄÄÅÄÄÄÅÄÄÄÅÄÄÄÅÄÄÄÅÄÄÄÅÄÄÄÅÄÄÄÅÄÄÄÅÄÄÄÅÄÄÄÅÄÄÄÅÄÄÄÅÄÄÄÅÄÄÄÅÄÄÄÅÄÄÄ¶
º#$nn*   
³00³BRK³JSR³RTI³RTS³nop³LDY³CPY³CPX³---³---³---³---³nopLDX³nopnop
º$nn,PC  
³10³BPL³BMI³BVC³BVS³BCC³BCS³BNE³BEQ³---³---³---³---³---³---³---³---º
º*       
³08³PHP³PLP³PHA³PLA³DEY³TAY³INY³INX³ASL³ROL³LSR³ROR³TXA³TAX³DEX³NOPº
º*       
³18³CLC³SEC³CLI³SEI³TYA³CLV³CLD³SED³nop³nop³nop³nop³TXS³TSX³nop³nopº
º$nn     
³04³nop³BIT³nop³nop³STY³LDY³CPY³CPX³ASL³ROL³LSR³ROR³STX³LDX³DEC³INCº
º$nn,X   
³14³nop³nop³nop³nop³STY³LDY³nop³nop³ASL³ROL³LSR³ROR³STX³LDX³DEC³INCº
º$nnnn   
³0C³nop³BIT³JMP³NDJ³STY³LDY³CPY³CPX³ASL³ROL³LSR³ROR³STX³LDX³DEC³INCº
º$nnnn,X 
³1C³nop³nop³nop³nop³shyLDY³nop³nop³ASL³ROL³LSR³ROR³shxLDX³DEC³INCº
º($nn,X) 
³01³ORA³AND³EOR³ADC³STA³LDA³CMP³SBC³slo³rla³sre³rra³sax³lax³dcp³isbº
º($nn),Y 
³11³ORA³AND³EOR³ADC³STA³LDA³CMP³SBC³slo³rla³sre³rra³shalax³dcp³isbº
º#$nn    
³09³ORA³AND³EOR³ADC³nop³LDA³CMP³SBC³ancancasrarranelxasbxsbcº
º$nnnn,Y 
³19³ORA³AND³EOR³ADC³STA³LDA³CMP³SBC³slo³rla³sre³rra³shslasdcp³isbº
º$nn     
³05³ORA³AND³EOR³ADC³STA³LDA³CMP³SBC³slo³rla³sre³rra³sax³lax³dcp³isbº
º$nn,X   
³15³ORA³AND³EOR³ADC³STA³LDA³CMP³SBC³slo³rla³sre³rra³sax³lax³dcp³isbº
º$nnnn   
³0D³ORA³AND³EOR³ADC³STA³LDA³CMP³SBC³slo³rla³sre³rra³sax³lax³dcp³isbº
º$nnnn,X 
³1D³ORA³AND³EOR³ADC³STA³LDA³CMP³SBC³slo³rla³sre³rra³shalax³dcp³isbº
ÈÍÍÍÍÍÍÍÍÏÍÍÏÍÍÍÏÍÍÍÏÍÍÍÏÍÍÍÏÍÍÍÏÍÍÍÏÍÍÍÏÍÍÍÏÍÍÍÏÍÍÍÏÍÍÍÏÍÍÍÏÍÍÍÏÍÍÍÏÍÍÍÏÍÍÍ¼

+-------------+
|table 2 notes|
+-------------+
    unusual operation (see "NMOS 65xx Instruction Set" document for 
details)
    jams machine rarely
---  jams machine

*: The first clock of any instruction is forced to read the next program 
counter address value into an internal 6502 temp reg, since any 6502 address 
has to be calculated one cycle before it can be accessed (the 6502's opcode 
fetch microcode cycles always increment & select the program counter as the 
next address to appear on the bus). For implied instructions, this means 
that the next instruction's opcode byte is loaded into an internal temp reg, 
but _not_ into the instruction register, which is where it would need to be 
to execute the instruction in just one clock. As a result, no 6502 
instructions are less than two clocks long.

*: JSR uses 2 byte immediate. The first immediate byte is read into a 6502 
temp reg on the first clock, then PC is pushed onto the stack. After, the 
second immediate byte is read in & transfered to PCH, simultaniously while 
loading PCL with the temp reg contents.

-JSR has a latency of six cycles, which includes one which seems to be a 
completely dead cycle. I think that this is because the 6502 is reusing the 
BRK microcode to perform the JSR.

-all instructions where any $nn,X and $nnnn,X addressing mode rows intersect 
with opcode columns 82 and A2, use the Y register for indexing.

-lowercase instructions are undocumented. However, most of them are 
basically composed of replacing the last microcode cycle of an instruction 
from the corresponding read-modify-write group (shift/inc/dec) column, with 
one from the load-execute group (and,ora,adc,etc.) column. The reason they 
can be combined like this, is because memory-based read-modify-write ALU 
operations don't do any special work on the last clock cycle of the 
instruction (the next instruction opcode fetch cycle), as the register-based 
ones do. This is why it's possible to perform two ALU functions in one 
instruction with the same latency as regular 6502 read-modify-write 
instructions, and thus makes the undocumented instructions highly efficient 
to use.


index adjust
------------
11001000  inc y
bit 6 =0, dec y
bit 5 =1, inc x
bit 1 =1, dec x


********************************
*Introduction to sound channels*
********************************
Thanks to Matthew Conte, Kentaro Ishihara, Goroh, Memblers, FluBBa, Izumi, 
Chibi-Tech, Quietust, SnowBro, Bananmos, and many others for their time and 
help on and off the NESdev mailing list, and the Membled Messageboards, in 
order to make the sound information here as accurate as possible.

The 2A03 (NES's integrated CPU) has 4 internal channels to it that have the 
ability to generate semi-analog sound, for musical playback purposes. These 
channels are 2 rectangle wave channels, one triangle wave channel, and a 
random wavelength channel. A fifth sound channel capable of playing samples 
based in the 6502's memory map, or fed directly to it in 7-bit unsigned PCM 
form, is also available.

Note that this document only details NTSC-related timing data for the sound 
channels, but here's some information in regards to that. "Apparently, 
timing differences between NTSC & PAL versions of the DMC exist only to 
ensure that the outputted sound is the same on either platform. this means 
that the difference between NTSC & PAL DMC timing, can simply be determined 
by comparing the CPU clock ratios of the 2 platforms."

After 2A03 reset, the sound channels are unavailable for playback during the 
first 2048 CPU clocks.


+--------------+
|Channel basics|
+--------------+
Each channel has different characteristics to it that make up it's 
operation. All listed frequencies assume that the 2A03 is being clocked with 
a 21.48 MHz signal.

The rectangle channel(s) have the ability to generate a rectangle wave 
frequency in the range of 54.6 Hz to 12.4 KHz. It's key features are 
frequency sweep abilities, and output duty cycle adjustment (square waves 
are also possible).

The triangle wave channel has the ability to generate an output triangle 
wave with a resolution of 4-bits (16 steps), in the range of 27.3 Hz to 55.9 
KHz. The key features this channel has is it's analog triangle wave output, 
and it's linear counter, which offers improved time resolution over the 
conventional length counter found in the same channel.

The random wavelength channel prodces waves of lengths in integer multiples 
inbetween 1 and 16 of 1-of-16 predefined base wavelengths. This results in 
the ability for this channel to be suitable for all kinds of noisey sound 
effect simulations. Output frequencys can range anywhere from 29.3 Hz to 447 
KHz. It's key feature is it's 15-bit shift register-based random number 
generator, which has two operational modes.

The delta modulation channel (DMC) is a complex series of digital counters 
and registers used to produce pretty decent-sounding analog audio. It's 
primary function is to play "samples" from memory, and have an internal 
counter connected to a digital to analog converter (DAC) updated 
accordingly. The channel is able to be assigned a pointer to a chunk of 
memory to be played. At timed intervals, the DMC will halt the 6502 for *2 
clock cycles to retrieve the sample to be played. This method of playback 
will be refered to here on as direct memory access (DMA) playback. Another 
method of playback known as pulse code modulation (PCM) is available by the 
channel, which requires the constant updating of one of the DMC's 
memory-mapped registers.

*: Goroh has quietly mentioned that a DMC DMA byte fetch phase takes 2 CPU 
clock cycles. However, I haven't confirmed this, and it is my belief that 
Nintendo would not design such a sloppy DMA unit; a 1 clock cycle DMA fetch 
would sound more logical. At best, this information should be taken with a 
grain of salt.


**********************************
*Low frequency programmable timer*
**********************************
The 2A03 has an internal programmable timer/counter, which is known as the 
frame counter. The purpose of it is to generate the various low frequency 
signals (60, 120, 240 Hz, and 48, 96, 192 Hz) required to clock several of 
the sound hardware's counters. It also has the ability to generate IRQ's.

The smallest unit of timing the frame counter operates around is 240Hz; all 
other frequencies are generated by multiples of this base frequency. An 
internal clock edge divider of 14915 off the 2A03's PHI2 line is used to get 
240Hz.


+-----------------------+
|Frame counter operation|
+-----------------------+
Depending on the status of $4017.7 (described later), the frame counter will 
follow 2 different count sequences. These sequences determine when sound 
hardware counters will be clocked, and is generally chosen in accordance 
with the target PPU type (i.e., NTSC or PAL) that an NES game is expected to 
run on. The sequences are initialized immediately following any write to 
$4017.

$4017.7  sequence
-------  --------
0        4, 0,1,2,3, 0,1,2,3,..., etc.
1        0,1,2,3,4, 0,1,2,3,4,..., etc.

During count sequences 0..3, the linear (triangle) and envelope decay 
(rectangle & noise) counters recieve a clock for each count. This means that 
both these counters are clocked once immediately after $4017.7 is written 
with a value of 1.

Count sequences 1 & 3 clock (update) the frequency sweep (rectangle), and 
length (all channels) counters. Even though the length counter's smallest 
unit of time counting is a frame, it seems that it is actually being clocked 
twice per frame. That said, you can consider the length counters to contain 
an extra stage to divide this clock signal by 2.

No aforementioned sound hardware counters are clocked on count sequence #4. 
You should now see how this causes the 96, and 192 Hz signals to be 
generated when $4017.7=1.

The rest of the document will describe the operation of the sound channels 
using the $4017.7=0 frequencies (60, 120, and 240 Hz). For $4017.7=1 
operation, replace those frequencies with 48, 96, and 192 Hz (respectively).


*********************************
*2A03 internal hardware port map*
*********************************
The sound hardware internal to the 2A03 has been designated these special 
memory addresses in the 6502's memory map.

$4000-$4003	Rectangle wave 1
$4004-$4007	Rectangle wave 2 (nearly identical to first)
$4008-$400B	Triangle
$400C-$400F	Noise
$4010   	DMC play mode and DMA frequency
$4011   	DMC delta counter
$4012   	DMC play code's starting address
$4013   	DMC length of play code
$4014   	transfer 256 bytes from written page to $2004
$4015r  	Channel enable / length/frame counter status
$4017   	frame counter control

Note: $4015 is the only R/W register here. All others do not respond to read 
cycles. Reads from $4016 and $4017 are decoded inside the 2A03, and those 
signals are available externally. Writes to bits D0-D2 of $4016 updates an 
internal 3-bit latch, with the status of those bits available externally.


+--------------+
|Register set 1|
+--------------+
$4000(rct1)/$4004(rct2)/$400C(noise) bits
---------------------------------------
0-3	volume / envelope decay rate
4	envelope decay disable
5	length counter clock disable / envelope decay looping enable
6-7	duty cycle type (unused on noise channel)

$4008(tri) bits
---------------
0-6	linear counter load register
7	length counter clock disable / linear counter start


+--------------+
|Register set 2|
+--------------+
$4001(rct1)/$4005(rct2) bits
--------------------------
0-2	right shift amount
3	decrease / increase (1/0) wavelength
4-6	sweep update rate
7	sweep enable

$4009(tri)/$400D(noise) bits
----------------------------
0-7	unused


+--------------+
|Register set 3|
+--------------+
$4002(rct1)/$4006(rct2)/$400A(Tri) bits
-------------------------------------
0-7	8 LSB of wavelength

$400E(noise) bits
-----------------
0-3	playback sample rate
4-6	unused
7	random number type generation


+--------------+
|Register set 4|
+--------------+
$4003(rct1)/$4007(rct2)/$400B(tri)/$400F(noise) bits
--------------------------------------------------
0-2	3 MS bits of wavelength (unused on noise channel)
3-7	length counter load register


+---------------------------------------+
|$4010 - DMC Play mode and DMA frequency|
+---------------------------------------+
This register is used to control the frequency of the DMA fetches, and to 
control the playback mode.

Bits
----
6-7	this is the playback mode.

	00 - play DMC sample until length counter reaches 0 (see $4013)
	x1 - loop the DMC sample (x = immaterial)
	10 - play DMC sample until length counter reaches 0, then generate a CPU 
IRQ

Looping (playback mode "x1") will have the chunk of memory played over and 
over, until the channel is disabled (via $4015). In this case, after the 
length counter reaches 0, it will be reloaded with the calculated length 
value of $4013.

If playback mode "10" is chosen, an interrupt will be dispatched when the 
length counter reaches 0 (after the sample is done playing). There are 2 
ways to acknowledge the DMC's interrupt request upon recieving it. The first 
is a write to this register ($4010), with the MSB (bit 7) cleared (0). The 
second is any write to $4015 (see the $4015 register description for more 
details).

If playback mode "00" is chosen, the sample plays until the length counter 
reaches 0. No interrupt is generated.

5-4	appear to be unused

3-0	this is the DMC frequency control. Valid values are from 0 - F. The 
value of this register determines how many CPU clocks to wait before the DMA 
will fetch another byte from memory. The # of clocks to wait -1 is initially 
loaded into an internal 12-bit down counter. The down counter is then 
decremented at the frequency of the CPU. The channel fetches the next DMC 
sample byte when the count reaches 0, and then reloads the count. This 
process repeats until the channel is disabled by $4015, or when the length 
counter has reached 0 (if not in the looping playback mode). The exact 
number of CPU clock cycles is as follows:

value	clocks  octave  scale
-----	------  ------  -----
F	1B0	8	C
E	240	7	G
D	2A8	7	E
C	350	7	C
B	400	6	A
A	470	6	G
9	500	6	F
8	5F0	6	D
7	6B0	6	C
6	710	5	B
5	7F0	5	A
4	8F0	5	G
3	A00	5	F
2	AA0	5	E
1	BE0	5	D
0	D60	5	C

The octave and scale values shown represent the DMC DMA clock cycle rate 
equivelant. These values are merely shown for the music enthusiast 
programmer, who is more familiar with notes than clock cycles.

Every fetched byte is loaded into a internal 8-bit shift register. The shift 
register is then clocked at 8x the DMA frequency (which means that the CPU 
clock count would be 1/8th that of the DMA clock count), or shifted at +3 
the octave of the DMA (same scale). The data shifted out of the register is 
in serial form, and the least significant bit (LSB, or bit 0) of the fetched 
byte is the first one to be shifted out (then bit 1, bit 2, etc.).

The bits shifted out are then fed to the UP/DOWN control pin of the internal 
delta counter, which will effectively have the counter increment it's 
retained value by one on "1" bit samples, and decrement it's value by one on 
"0" bit samples. This effectively clocks the counter once (up or down) for 
every shift register clock.

The counter is only 6 bits in size, and has it's 6 outputs tied to the 6 MSB 
inputs of a 7 bit DAC. The analog output of the DAC is then what you hear 
being played by the DMC.

Wrap around counting is not allowed on this counter. Instead, a "clipping" 
behaviour is exhibited. If the internal value of the counter has reached 0, 
and the next bit sample is a 0 (instructing a decrement), the counter will 
take no action. Likewise, if the counter's value is currently at -1 
(111111B, or 03FH), and the bit sample to be played is a 1, the counter will 
not increment.


+---------------------------------------+
|$4011 - DMC Delta counter load register|
+---------------------------------------+
bits
----
7	appears to be unused
1-6	the load inputs of the internal delta counter
0	LSB of the DAC

A write to this register effectively loads the internal delta counter with a 
6 bit value. Bit 0 is connected directly to the LSB (bit 0) of the DAC, and 
has no effect on the internal delta counter. Bit 7 appears to be unused.

This register can be used to output direct 7-bit digital PCM data to the 
DMC's audio output. To use this register for PCM playback, the programmer 
would be responsible for making sure that this register is updated at a 
constant rate (therefore it is completely user-definable). A practical 
update rate for this register would be every scanline (113.67 CPU clocks) 
for a 15.7458 KHz playback rate.

Another use of this register (although unrelated to DMC playback) has been 
to somewhat control the volume of the Triangle & Noise sound channel 
outputs. Please see NESSOUND.TXT for more information.

On 2A03 reset, all 7 used bits of $4011 are reset to 0, the DMC's IRQ flag 
is cleared (disabled), and the channel is disabled. All other registers will 
remain unmodified.


+---------------------------------+
|$4012 - DMC address load register|
+---------------------------------+
This register contains the initial address where the DMC is to fetch samples 
from memory for playback. The effective address value is $4012 shl 6 or 
0C000H. This register is connected to the load pins of the internal DMA 
address pointer register (counter). The counter is incremented after every 
DMA byte fetch. The counter is 15 bits in size, and has addresses wrap 
around from $FFFF to $8000 (not $C000, as you might have guessed). The DMA 
address pointer register is reloaded with the initial calculated address, 
when the DMC is activated from an inactive state, or when the length counter 
has arrived at terminal count (count=0), if in the looping playback mode.


+---------------------------+
|$4013 - DMC length register|
+---------------------------+
This register contains the length of the chunk of memory to be played by the 
DMC, and it's size is measured in bytes. The value of $4013 shl 4 is loaded 
into a 12 bit internal down counter, dubbed the length counter. The length 
counter is decremented after every DMA fetch, and when it arrives at 0, the 
DMC will take action(s) based on the 2 MSB of $4010. This counter will be 
loaded with the current calculated address value of $4013 when the DMC is 
activated from an inactive state. Because the value that is loaded by the 
length counter is $4013 shl 4, this effectively produces a calculated byte 
sample length of $4013 shl 4 + 1 (i.e. if $4013=0, sample length is 1 byte 
long; if $4013=FF, sample length is $FF1 bytes long).


+-----------------------------------------------------+
|$4014 - transfer 256 bytes from written page to $2004|
+-----------------------------------------------------+
As the name implies, writing to this port will cause the written value to be 
used as the high 8-bits of the source 6502 address, and transfer 256 
individual bytes from the source address maintained by an internal 8-bit up 
counter, to $2004, a hardcoded address where the 2C02 (the NES's PPU) is 
normally mapped in. Page transfers take 512 CPU clock cycles, but details on 
when it starts are not clear. "The CPU either fetches the first byte of the 
next instruction, and then begins DMA, or fetches and executes the next 
instruction, and then begins DMA".


+-------------------------------------------------------------+
|$4015 - DMC/IRQ/length counter status/channel enable register|
+-------------------------------------------------------------+
read
----
0	rectangle wave channel 1 length counter status
1	rectangle wave channel 2 length counter status
2	triangle wave channel length counter status
3	noise channel length counter status
4	DMC is currently enabled (playing a stream of samples)
5	unknown
6	frame IRQ status (active when set)
7	DMC's IRQ status (active when set)

write
-----
0	rectangle wave channel 1 enable
1	rectangle wave channel 2 enable
2	triangle wave channel enable
3	noise channel enable
4	enable/disable DMC (1=start/continue playing a sample;0=stop playing)
5-7	unknown


When an IRQ goes off inside the 2A03, Bit 7 of $4015 can tell the interrupt 
handler if it was caused by the DMC hardware or not. This bit will be set 
(1) if the DMC is responsible for the IRQ. Of course, if your program has no 
other IRQ-generating hardware going while it's using the DMC, then reading 
this register is not neccessary upon IRQ generation. Note that reading this 
register will NOT clear bit 7 (meaning that the DMC's IRQ will still NOT be 
acknowledged). Also note that if the 2 MSBs of $4010 are not set to 10, no 
IRQ will be generated, and bit 7 will always be 0.

Upon generation of an IRQ, to let the DMC know that the software has 
acknowledged the /IRQ (and to reset the DMC's internal IRQ flag), any write 
out to $4015 will reset the flag, or a write out to $4010 with the MSB set 
to 0 will do. These practices should be performed inside the IRQ handler 
routine. To replay the same sample that just finished, all you need to do is 
just write a 1 out to bit 4 of $4015.

Bit 4 of $4015 reports the real-time status of the DMC. A returned value of 
1 denotes that the channel is currently playing a stream of samples. A 
returned value of 0 indicates that the channel is inactive. If the 
programmer needed to know when a stream of samples was finished playing, but 
didn't want to use the IRQ generation feature of the DMC, then polling this 
bit would be a valid option.

Writing a value to $4015's 4th bit has the effect of enabling the channel 
(start, or continue playing a stream of samples), or disabling the channel 
(stop all DMC activity). Note that writing a 1 to this bit while the channel 
is currently enabled, will have no effect on counters or registers internal 
to the DMC.

The conditions that control the time the DMC will stay enabled are 
determined by the 2 MSB of $4010, and register $4013 (if applicable).

Note that all 5 writable bits in $4015 will be set to 0 upon 2A03 reset.


+-----------------------------------+
|$4017 - Low frequency timer control|
+-----------------------------------+
Writes to register $4017 control operation of both the clock divider, and 
the frame counter.

- Any write to $4017 resets both the frame counter, and the clock divider. 
Sometimes, games will write to this register in order to synchronize the 
sound hardware's internal timing, to the sound routine's timing (usually 
tied into the NMI code). The frame IRQ frequency is slightly smaller than 
the PPU's vertical retrace frequency, so you can see why games would desire 
this syncronization.

- bit 6: enable frame IRQ's (when zero).

- bit 7: NTSC/PAL framerate switch (0/1). This bit controls the frame 
counter's divide rate. Every time the counter cycles (reaches terminal count 
(0)), a frame IRQ will be generated, if enabled by clearing bit 6 of $4017. 
$4015.6 holds the status of the frame counter IRQ; it will be set if the 
frame counter is responsible for the interrupt.

$4017.7 divider  frame IRQ freq.
------- -------  ---------------
0       4        60
1       5        48

On 2A03 reset, both bits of $4017 (6 & 7) will be cleared, enabling frame 
IRQ's off the hop. The reason why the existence of frame IRQ's are generally 
unknown is because the 6502's maskable interrupt is disabled on reset, and 
this blocks out the frame IRQ's. Most games don't use any IRQ-generating 
hardware in general, therefore they don't bother enabling maskable 
interrupts.

Note that the IRQ line will be held down by the frame counter until it is 
acknowledged (by reading $4015). Before this, the 6502 will generate an IRQ 
*every* time interrupts are enabled (either by CLI or RTI), since the IRQ 
design on the 6502 is level-triggered, and not edge. So bottom line: if 
you're going to enable interrupts in an IRQ handler, make sure you've 
serviced the device responsible for the IRQ first.


*******************************************
*microarchitecture of basic sound channels*
*******************************************
This section will describe the internal components that make up the basic 
sound channels.


Device                        Triangle Noise  Rectangle
------                        -------- ------ ---------
triangle step generator              X
linear counter                       X
programmable timer                   X      X      X
length counter                       X      X      X
4-bit DAC                            X      X      X
volume/envelope decay unit                  X      X
sweep unit                                         X
duty cycle generator                               X
wavelength converter                        X
random number generator                     X


+-------------------------+
| Triangle step generator |
+-------------------------+
This is a 5-bit, single direction counter, and it is only used in the 
triangle channel. Each of the 4 LSB outputs of the counter lead to one input 
on a corresponding mutually exclusive XNOR gate. The 4 XNOR gates have been 
strobed together, which results in the inverted representation of the 4 LSB 
of the counter appearing on the outputs of the gates when the strobe is 0, 
and a non-inverting action taking place when the strobe is 1. The strobe is 
naturally connected to the MSB of the counter, which effectively produces on 
the output of the XNOR gates a count sequence which reflects the scenario of 
a near- ideal triangle step generator (D,E,F,F,E,D,...,2,1,0,0,1,2,...). At 
this point, the outputs of the XNOR gates will be fed into the input of a 
4-bit DAC.

This 5-bit counter will be halted whenever the Triangle channel's length or 
linear counter contains a count of 0. This results in a "latching" 
behaviour; the counter will NOT be reset to any definite state.

On 2A03 reset, this counter is loaded with 0.

The counter's clock input is connected directly to the terminal count output 
pin of the 11-bit programmable timer in the triangle channel. As a result of 
the 5-bit triangle step generator, the output triangle wave frequency will 
be 32 times less than the frequency of the triangle channel's programmable 
timer is set to generate.


+----------------+
| Linear counter |
+----------------+


+--------------------+
| Programmable timer |
+--------------------+
The programmable timer is a 11-bit presettable down counter, and is found in 
the rectangle, triangle, and noise channel(s). The bit assignments are as 
follows:

$4002(rct1)/$4006(rct2)/$400A(Tri) bits
-------------------------------------
0-7	represent bits 0-7 of the 11-bit wavelength

$4003(rct1)/$4007(rct2)/$400B(Tri) bits
-------------------------------------
0-2	represent bits 8-A of the 11-bit wavelength

Note that on the noise channel, the 11 bits are not available directly. See 
the wavelength converter section, for more details.

The counter has automatic syncronous reloading upon terminal count 
(count=0), therefore the counter will count for N+1 (N is the 11-bit loaded 
value) clock cycles before arriving at terminal count, and reloading. This 
counter will typically be clocked at the 2A03's internal 6502 speed (1.79 
MHz), and produces an output frequency of 1.79 MHz/(N+1). The terminal 
count's output spike length is typically no longer than half a CPU clock. 
The TC signal will then be fed to the appropriate device for the particular 
sound channel (for rectangle, this terminal count spike will lead to the 
duty cycle generator. For the triangle, the spike will be fed to the 
triangle step generator. For noise, this signal will go to the random number 
generator unit).


+----------------+
| Length counter |
+----------------+
The length counter is found in all sound channels. It is essentially a 7-bit 
down counter, and is conditionally clocked at a frequency of 60 Hz.

When the length counter arrives at a count of 0, the counter will be stopped 
(stay on 0), and the appropriate channel will be silenced.

The length counter clock disable bit, found in all the channels, can also be 
used to halt the count sequence of the length counter for the appropriate 
channel, by writing a 1 out to it. A 0 condition will permit counting 
(unless of course, the counter's current count = 0). Location(s) of the 
length counter clock disable bit:

$4000(rct1)/$4004(rct2)/$400C(noise) bits
---------------------------------------
5	length counter clock disable

$4008(tri) bits
---------------
7	length counter clock disable

To load the length counter with a specified count, a write must be made out 
to the length register. Location(s) of the length register:

$4003(rct1)/$4007(rct2)/$400B(tri)/$400F(noise) bits
--------------------------------------------------
3-7	length

The 5-bit length value written, determines what 7-bit value the length 
counter will start counting from. A conversion table here will show how the 
values are translated.

	+-----------------------+
	|	bit3=0		|
	+-------+---------------+
	|	|frames		|
	|bits	+-------+-------+
	|4-6	|bit7=0	|bit7=1	|
	+-------+-------+-------+
	|0	|05	|06	|
	|1	|0A	|0C	|
	|2	|14	|18	|
	|3	|28	|30	|
	|4	|50	|60	|
	|5	|1E	|24	|
	|6	|07	|08	|
	|7	|0D	|10	|
	+-------+-------+-------+

	+---------------+
	|	bit3=1	|
	+-------+-------+
	|bits	|	|
	|4-7	|frames	|
	+-------+-------+
	|0	|7F	|
	|1	|01	|
	|2	|02	|
	|3	|03	|
	|4	|04	|
	|5	|05	|
	|6	|06	|
	|7	|07	|
	|8	|08	|
	|9	|09	|
	|A	|0A	|
	|B	|0B	|
	|C	|0C	|
	|D	|0D	|
	|E	|0E	|
	|F	|0F	|
	+-------+-------+

The length counter's real-time status for each channel can be attained. A 0 
is returned for a zero count status in the length counter (channel's sound 
is disabled), and 1 for a non-zero status. Here's the bit description of the 
length counter status register:

$4015(read)
-----------
0	length counter status of rectangle wave channel 1
1	length counter status of rectangle wave channel 2
2	length counter status of triangle wave channel
3	length counter status of noise channel
4	length counter status of DMC
5	unknown
6	frame IRQ status
7	IRQ status of DMC

Writing a 0 to the channel enable register will force the length counters to 
always contain a count equal to 0, which renders that specific channel 
disabled (as if it doesn't exist). Writing a 1 to the channel enable 
register disables the forced length counter value of 0, but will not change 
the count itself (it will still be whatever it was prior to the writing of 
1).

Bit description of the channel enable register:

$4015(write)
------------
0	enable rectangle wave channel 1
1	enable rectangle wave channel 2
2	enable triangle wave channel
3	enable noise channel
4	enable DMC channel
5-7	unknown


+-----------+
| 4-bit DAC |
+-----------+
This is just a standard 4-bit DAC with 16 steps of output voltage 
resolution, and is used by all 4 sound channels. On the 2A03, rectangle wave 
1 & 2 are mixed together, and are available via pin 1. Triangle & noise are 
available on pin 2.

These analog outputs require a negative current source, to attain linear 
symmetry on the various output voltage levels generated by the channel(s) 
(moreover, to get the sound to be audible). Instead of current sources, the 
NES uses external 100 ohm pull-down resistors. This results in the output 
waveforms having some linear asymmetry (i.e., as the desired output voltage 
increases on a linear scale, the actual outputted voltage increases less and 
less each step).

The side effect of this is that the DMC's 7-bit DAC port ($4011) is able to 
indirectly control the volume (somewhat) of both triangle & noise channels. 
While I have not measured the voltage asymmetery, others on the Membled 
Messageboards have posted their findings. The conclusion is that when $4011 
is 0, triangle & noise volume outputs are at maximum. When $4011 = 7F, the 
triangle & noise channel outputs operate at only 57% total volume. The odd 
thing is that a few games actually take advantage of this "volume" feature, 
and write values to $4011 in order to regulate the amplitude of the triangle 
wave channel's output.

The best circuit I've found to use for reproducing a signal coming out of 
either pin of the 2A03 as accurately and with as few components as possible, 
is to use a PNP transistor with it's emitter connected to the 2A03 audio 
source pin(s), it's base connected to a simple adjustable voltage source 
composed of a 500-2000 ohm potentiometer dropped across the +5VDC power 
supply, and a 5-10 K ohm resistor connected between the collector and 
ground. Retrieve the amplified audio off the collector (w/ resp. to ground), 
and adjust the potentiometer for desired volume. In a two-transistor circuit 
for stereo amplification, it's okay to use the same potentiometer, but it 
may be desirable to adjust one channel to be quieter than the other (though 
this is generally not neccessary).


+------------------------------+
| Volume / envelope decay unit |
+------------------------------+
The volume / envelope decay hardware is found only in the rectangle wave and 
noise channels.

$4000(rct1)/$4004(rct2)/$400C(noise)
----------------------------------
0-3	volume / envelope decay rate
4	envelope decay disable
5	envelope decay looping enable

When the envelope decay disable bit (bit 4) is set (1), the current volume 
value (bits 0-3) is sent directly to the channel's DAC. However, depending 
on certain conditions, this 4-bit volume value will be ignored, and a value 
of 0 will be sent to the DAC instead. This means that while the channel is 
enabled (producing sound), the output of the channel (what you'll hear from 
the DAC) will either be the 4-bit volume value, or 0. This also means that a 
4-bit volume value of 0 will result in no audible sound. These conditions 
are as follows:

- When hardware in the channel wants to disable it's sound output (like the 
length counter, or sweep unit (rectangle channels only)).

- On the negative portion of the output frequency signal coming from the 
duty cycle / random number generator hardware (rectangle wave channel / 
noise channel).

When the envelope decay disable bit is cleared, bits 0-3 now control the 
envelope decay rate, and an internal 4-bit down counter (hereon the envelope 
decay counter) now controls the channel's volume level. "Envelope decay" is 
used to describe the action of the channel's audio output volume starting 
from a certain value, and decreasing by 1 at a fixed (linear) rate (which 
produces a "fade-out" sounding effect). This fixed decrement rate is 
controlled by the envelope decay rate (bits 0-3). The calculated decrement 
rate is 240Hz/(N+1), where N is any value between $0-$F.

When the channel's envelope decay counter reaches a value of 0, depending on 
the status of the envelope decay looping enable bit (bit 5, which is shared 
with the length counter's clock disable bit), 2 different things will 
happen:

bit 5	action
-----	------
0	The envelope decay count will stay at 0 (channel silenced).
1	The envelope decay count will wrap-around to $F (upon the next clock 
cycle). The envelope decay counter will then continue to count down 
normally.

Only a write out to $4003/$4007/$400F will reset the current envelope decay 
counter to a known state (to $F, the maximum volume level) for the 
appropriate channel's envelope decay hardware. Otherwise, the envelope decay 
counter is always counting down (by 1) at the frequency currently contained 
in the volume / envelope decay rate bits (even when envelope decays are 
disabled (setting bit 4)), except when the envelope decay counter contains a 
value of 0, and envelope decay looping (bit 5) is disabled (0).


+------------+
| Sweep unit |
+------------+
The sweep unit is only found in the rectangle wave channels. The controls 
for the sweep unit have been mapped in at $4001 for rectangle 1, and $4005 
for rectangle 2.

The controls
------------
Bit 7   	when this bit is set (1), sweeping is active. This results in 
real-time increasing or decreasing of the the current wavelength value (the 
audible frequency will decrease or increase, respectively). The wavelength 
value in $4002/3 ($4006/7) is constantly read & updated by the sweep. 
Modifying the contents of $4002/3 will be immediately audible, and will 
result in the sweep now starting from this new wavelength value.

Bits 6-4	These 3 bits represent the sweep refresh rate, or the frequency at 
which $4002/3 is updated with the new calculated wavelength. The refresh 
rate frequency is 120Hz/(N+1), where N is the value written, between 0 and 
7.

Bit 3   	This bit controls the sweep mode. When this bit is set (1), sweeps 
will decrease the current wavelength value, as a 0 will increase the current 
wavelength.

Bits 2-0	These bits control the right shift amount of the new calculated 
sweep update wavelength. Code that shows how the sweep unit calculates a new 
sweep wavelength is as follows:

bit 3
-----
0	New = Wavelength + (Wavelength >> N)
1	New = Wavelength - (Wavelength >> N) (minus an additional 1, if using 
rectangle wave channel 1)

where N is the the shift right value, between 0-7.

Note that in decrease mode, for subtracting the 2 values:
1's compliment (NOT) is being used for rectangle wave channel 1
2's compliment (NEG) is being used for rectangle wave channel 2

This information is currently the only known difference between the 2 
rectangle wave channels.

On each sweep refresh clock, the Wavelength register will be updated with 
the New value, but only if all 3 of these conditions are met:

- bit 7 is set (sweeping enabled)
- the shift value (which is N in the formula) does not equal to 0
- the channel's length counter contains a non-zero value

Notes
-----
There are certain conditions that will cause the sweep unit to silence the 
channel, and halt the sweep refresh clock (which effectively stops sweep 
action, if any). Note that these conditions pertain regardless of any sweep 
refresh rate values, or if sweeping is enabled/disabled (via bit 7).

- an 11-bit wavelength value less than $008 will cause this condition
- if the sweep unit is currently set to increase mode, the New calculated 
wavelength value will always be tested to see if a carry (bit $B) was 
generated or not (if sweeping is enabled, this carry will be examined before 
the Wavelength register is updated) from the shift addition calculation. If 
carry equals 1, the channel is silenced, and sweep action is halted.


+----------------------+
| Duty cycle generator |
+----------------------+
The duty cycle generator takes the fequency produced from the 11-bit 
programmable timer, and uses a 4 bit counter to produce 4 types of duty 
cycles. The output frequency is then 1/16 that of the programmable timer. 
The duty cycle hardware is only found in the rectangle wave channels. The 
bit assignments are as follows:

$4000(rct1)/$4004(rct2)
---------------------
6-7	Duty cycle type

	duty (positive/negative)
val	in clock cycles
---	---------------
00	 2/14
01	 4/12
10	 8/ 8
11	12/ 4

Where val represents bits 6-7 of $4000/$4004.

This counter is reset when the length counter of the same channel is written 
to (via $4003/$4007).

The output frequency at this point will now be fed to the volume/envelope 
decay hardware.


+----------------------+
| Wavelength converter |
+----------------------+
The wavelength converter is only used in the noise channel. It is used to 
convert a given 4-bit value to an 11-bit wavelength, which then is sent to 
the noise's own programmable timer. Here is the bit descriptions:

$400E bits
----------
0-3	The 4-bit value to be converted

Below is a conversion chart that shows what 4-bit value will represent the 
11-bit wavelength to be fed to the channel's programmable timer:

value	octave	scale	CPU clock cycles (11-bit wavelength+1)
-----	------	-----	--------------------------------------
0	15	A	002
1	14	A	004
2	13	A	008
3	12	A	010
4	11	A	020
5	11	D	030
6	10	A	040
7	10	F	050
8	10	C	065
9	 9	A	07F
A	 9	D	0BE
B	 8	A	0FE
C	 8	D	17D
D	 7	A	1FC
E	 6	A	3F9
F	 5	A	7F2

Octave and scale information is provided for the music enthusiast programmer 
who is more familiar with notes than clock cycles.


+-------------------------+
| Random number generator |
+-------------------------+
The noise channel has a 1-bit pseudo-random number generator. It's based on 
a 15-bit shift register, and an exclusive or gate. The generator can produce 
two types of random number sequences: long, and short. The long sequence 
generates 32,767-bit long number patterns. The short sequence generates 
93-bit long number patterns. The 93-bit mode will generally produce higher 
sounding playback frequencys on the channel. Here is the bit that controls 
the mode:

$400E bits
----------
7	mode

If mode=0, then 32,767-bit long number sequences will be produced (32K 
mode), otherwise 93-bit long number sequences will be produced (93-bit 
mode).

The following diagram shows where the XOR taps are taken off the shift 
register to produce the 1-bit pseudo-random number sequences for each mode.

mode	    <-----
----	EDCBA9876543210
32K	**
93-bit	*     *

The current result of the XOR will be transferred into bit position 0 of the 
SR, upon the next shift cycle. The 1-bit random number output is taken from 
pin E, is inverted, then is sent to the volume/envelope decay hardware for 
the noise channel. The shift register is shifted upon recieving 2 clock 
pulses from the programmable timer (the shift frequency will be half that of 
the frequency from the programmable timer (one octave lower)).

On 2A03 reset, this shift register is loaded with a value of 1.


RP2A03E quirk
-------------
I have been informed that revisions of the 2A03 before "F" actually lacked 
support for the 93-bit looped noise playback mode. While the Famicom's 2A03 
went through 4 revisions (E..H), I think that only one was ever used for the 
front loading NES: "G". Other differences between 2A03 revisions are 
unknown.


EOF
