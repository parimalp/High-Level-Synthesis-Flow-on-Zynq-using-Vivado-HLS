# Creating a Processor System Lab

## Objectives

After completing this lab, you will be able to:

* Understand the steps and directives involved in creating an IP-XACT adapter from a synthesized
design in Vivado HLS
* Create a processor system using IP Integrator in Vivado
* Integrate the generated IP-XACT adapter into the created processor system

## The design

The design consists of a FIR filter to filter a 4 KHz tone added to CD quality (48 KHz) music. The
characteristic of the filter is as follows:

FS=48000 Hz  
FPASS1=2000 Hz  
FSTOP1=3800 Hz  
FSTOP2=4200 Hz  
FPASS2=6000 Hz  
APASS1=APASS2=1 dB  
ASTOP=60 dB

This lab requires you to develop a peripheral core of the designed filter that can be instantiated in a
processor system. The processor system will acquire a stereo music stream using an on-board CODEC
chip and I2C controller, process it through the designed filter (bandstop filter), and output back to the
headphone. **Note: You won't be able to validate this lab on the PYNQ-Z1 board as the board does not have Audio CODEC chip.**

## Steps

### Create a New Project 

#### Create a new project in Vivado HLS targeting xc7z020clg400-1 device

1. Select **Start > Xilinx Design Tools > Vivado HLS 2018.2**  

	A **Getting Started GUI** will appear.

2. In the Getting Started section, click on *Create New Project*. The **New Vivado HLS Project** wizard opens.

3. Click **Browse…** button of the *Location* field, browse to **{labs}\lab4**, and then click **OK**.

4. For *Project* Name, type **fir.prj** and click **Next**.

5. In the *Add/Remove Files* for the source files, type **fir** as the function name (the provided source file contains the function, to be synthesized, called fir).

6. Click the *Add Files…* button, select **fir.c** and **fir\_coef.dat** files from the **{labs}\lab4** folder, and then click **Open**.

7. Click **Next**.

8. In the *Add/Remove Files* for the testbench, click the *Add Files…* button, select **fir\_test.c** file from the **{labs}\lab4** folder and click **Open**.

9. Click **Next**.

10. In the *Solution Configuration* page, leave *Solution* Name field as solution1 and make sure the clock period as 10. Leave Uncertainty field blank.

11. Click on the Part’s Browse button and using the *Parts Specify* option, select **xc7z020clg400-1**.

12. Click **Finish**.

	You will see the created project in the Explorer view. Expand various sub-folders to see the entries under each sub-folder.

13. Double-click on the *fir.c* under the source folder to open its content in the information pane.
	<p align="center">
	<img src ="./images/lab4/Figure1.png">
	</p>
	<p align = "center">
	<i>The design under consideration</i>
	</p>
	
	The FIR filter expects **x** as a sample input and pointer to the computed sample out **y**. Both of them are defined of data type data\_t. The coefficients are loaded in array **c** of type coef\_t from the file called *fir\_coef.dat* located in the current directory. The sequential algorithm is applied and accumulated value (sample out) is computed in variable acc of type acc\_t.

14. Double-click on the **fir.h** in the outline tab to open its content in the information pane.
	<p align="center">
	<img src ="./images/lab4/Figure2.png">
	</p>
	<p align = "center">
	<i>The header file</i>
	</p>

	The header file includes ap\_cint.h so user defined data width (of arbitrary precision) can be used. It also defines number of taps (N), number of samples to be generated (in the testbench), and data types coef\_t, data\_t, and acc\_t. The coef\_t and data\_t are short (16 bits). Since the algorithm iterates (multiply and accumulate) over 59 taps, there is a possibility of bit growth of 6 bits and hence acc\_t is defined as int38. Since the acc\_t is bigger than sample and coefficient width, they have to cast before being used (like in lines 16, 18, and 21 of fir.c).

15. Double-click on the **fir\_test.c** under the testbench folder to open its content in the information pane.

	Notice that the testbench opens fir\_impulse.dat in write mode, and sends an impulse (first sample
	being 0x8000.

### Run C Simulation

#### Run C simulation to observe the expected output.

1. Select **Project > Run C Simulation** or click on the button from the tools bar buttons, and click **OK** in the C Simulation Dialog window.

	The testbench will be compiled using apcc compiler and csim.exe file will be generated. The csim.exe will then be executed and the output will be displayed in the console view.
	<p align="center">
	<img src ="./images/lab4/Figure3.png">
	</p>
	<p align = "center">
	<i>Initial part of the generated output in the Console view</i>
	</p>

	You should see the filter coefficients being computed.

### Synthesize the Design

#### Synthesize the design with the defaults. View the synthesis results and answer the question listed in the detailed section of this step.

1. Select **Solution > Run C Synthesis > Active Solution** to start the synthesis process.

2. When synthesis is completed, several report files will become accessible and the Synthesis
Results will be displayed in the information pane.

3. The *Synthesis Report* shows the performance and resource estimates as well as estimated
latency in the design.

4. Using scroll bar on the right, scroll down into the report and answer the following question.

	**Question 1**

	Estimated clock period:  
	Worst case latency:  
	Number of DSP48E used:  
	Number of BRAMs used:  
	Number of FFs used:  
	Number of LUTs used:  

5. The report also shows the top-level interface signals generated by the tools.
	<p align="center">
	<img src ="./images/lab4/Figure4.png">
	</p>
	<p align = "center">
	<i>Generated interface signals</i>
	</p>

	You can see the design expects x input as 16-bit scalar and outputs y via pointer of the 16-bit
    data. It also has ap\_vld signal to indicate when the result is valid.

#### Add PIPELINE directive to the loop and re-synthesize the design. View the synthesis results.

1. Make sure that the *fir.c* is open in the information view.

2. Select the **Directive** tab, and apply the **PIPELINE** directive to the loop.

3. Select **Solution > Run C Synthesis > Active Solution** to start the synthesis process.

4. When synthesis is completed, the *Synthesis Results* will be displayed in the information pane.

5. Note that the latency has reduced to 62 clock cycles. The DSP48 and BRAM consumption
remains same; however, LUT and FF consumptions have slightly increased.

### Run RTL/C CoSimulation

#### Run the RTL/C Co-simulation, selecting Verilog. Verify that the simulation passes.

1. Select **Solution > Run C/RTL Co-simulation** or click on the button to open the dialog box so the desired simulations can be run.

2. Select the *Verilog* option and click **OK**.

	The Co-simulation will run, generating and compiling several files, and then simulating the design. In the console window you can see the progress. When done the RTL Simulation Report shows that it was successful and the latency reported was 62.

### Setup IP-XACT Adapter

#### Add INTERFACE directive to create AXI4LiteS adapters so IP-XACT adapter can be generated during the RTL Export step.

1. Make sure that **fir.c** file is open and in focus in the information view.

2. Select the **Directive** tab.

3. Right-click *x*, and click on **Insert Directive…**.

4. In the Vivado HLS Directive Editor dialog box, select **INTERFACE** using the drop-down button.

5. Click on the button beside *mode (optional)*. Select **s\_axilite**.

6. In the *bundle (optional)* field, enter **fir\_io** and click **OK**.
    <p align="center">
    <img src ="./images/lab4/Figure5.png">
    </p>
    <p align = "center">
    <i>Selecting the AXI4LiteS adapter and naming bundle</i>
    </p>
7. Similarly, apply the **INTERFACE** directive (including bundle) to the *y* output.
    <p align="center">
    <img src ="./images/lab4/Figure6.png" width="60%" height="80%"/>
    </p>
    <p align = "center">
    <i>Applying bundle to assign y output to AXI4Lite adapter</i>
    </p>
8. Apply the **INTERFACE** directive to the top-level module *fir* to include ap\_start, ap\_done, and ap\_idle signals as part of bus adapter (the variable name shown will be return). Include the bundle information too.

	<p align="center">
	<img src ="./images/lab4/Figure7.png">
	</p>
	<p align = "center">
	<i>Applying bundle to assign function control signals to AXI4Lite adapter</i>
	</p>
   
	Note that the above steps will create address maps for x, y, ap\_start ap\_valid, ap\_done, and ap\_idle, which can be accessed via software. Alternately, ap\_start, ap\_valid, ap\_done, ap\_idle signals can be generated as separate ports on the core by not applying RESOURCE directive to the top-level module fir. These ports will then have to be connected in a processor system using available GPIO IP.

### Generate IP-XACT Adapter

#### Re-synthesize the design as directives have been added. Run the RTL Export to generate the IP-XACT adapter.

1. Since the directives have been added, it is safe to re-synthesize the design. Select **Solution > Run C Synthesis > Active Solution**

	Check the Interface summary at the bottom of the Synthesis report to see the interface that has been created.

2. Once the design is synthesized, select **Solution > Export RTL** to open the dialog box so the desired IP can be generated.

	An **Export RTL Dialog** box will open.
     <p align="center">
     <img src ="./images/lab4/Figure8.png">
     </p>
     <p align = "center">
     <i>Export RTL Dialog</i>
     </p>

3. Click **OK** to generate the IP-XACT adapter.

4. When the run is completed, expand the **impl** folder in the Explorer view and observe various generated directories, such as ip, misc, verilog and vhdl.

	<p align="center">
	<img src ="./images/lab4/Figure9.png">
	</p>
	<p align = "center">
	<i>IP-XACT adapter generated</i>
	</p>

	Expand the ip directory and observe several files and sub-directories. One of the sub-directory of interest is the drivers directory which consists of header, c, tcl, mdd, and makefile files. Another file of interest is the zip file, which is the ip repository file that can be imported in an IP Integrator design

	<p align="center">
	<img src ="./images/lab4/Figure10.png">
	</p>
	<p align = "center">
	<i>Adapter’s drivers directory</i>
	</p>

5. Close Vivado HLS by selecting **File > Exit**.

### Create a Vivado Project

#### Launch Vivado Tcl Shell and run the provided tcl script to create an initial system targeting the PYNQ-Z2 board.

1. Open *Vivado Tcl Shell* by selecting **Start > Xilinx Design Tools > Vivado 2018.2 Tcl Shell**

2. In the shell window, change the directory to **{labs}/lab4** using the *cd* command.

3. Run the provided script file to create an initial system having zed\_audio\_ctrl and GPIO peripherals by typing the following command:
    
	*source pynq\_z2\_audio\_project\_create.tcl*
    
    The script will be run and the initial system, shown below, will be created.

    <p align="center">
    <img src ="./images/lab4/Figure11.png">
    </p>
    <p align = "center">
    <i>Block design made for Pynq</i>
    </p>

#### Add the HLS IP to the IP Catalog

1. Select **Flow Navigator > Project Manager > Settings**

2. Expand **IP > Repository** in the left pane.

3. Click the **+** button (The lab4/ip\_repo directory has already been added). Browse to
**{labs}\lab4\fir.prj\solution1\impl\ip** and click **Select**.

   The directory will be scanned and added in the IP Repositories window, and one IP entry will be detected.

4. Click **OK**.
    <p align="center">
    <img src ="./images/lab4/Figure12.png">
    </p>
    <p align = "center">
    <i>Setting path to IP Repositories</i>
    </p>

5. Click **OK** to accept the settings

#### Instantiate fir\_top core twice, one for each side channel, into the processing system naming the instances as fir\_left and fir\_right.

1. Click the *Add IP* icon (plus sign icon) and search for **Fir** in the catalog by typing **Fir** and double-click on the **Fir** entry to add an instance.

     Notice that the added IP has HLS logo in it indicating that this was created by Vivado HLS.

2. Select the added instance in the diagram, and change its instance name to **fir\_left** by typing it in the *Name* field of the *Block Properties* form in the left.

3. Similarly, add another instance of the HLS IP, and naming it **fir\_right**.

4. Click on **Run Connection Automation**, and select **All Automation**.

5. Click on **/fir\_left/s\_axi\_fir\_io** and **/fir\_right/s\_axi\_fir\_io**. and verify that they will both be   connected to the M\_AXI\_GP0, and click **OK**.

#### Enable the PS-PL Interrupt ports > IRQ\_F2P ports. Add an instance of concat IP with two single-bit input ports. Connect input ports to the interrupt ports of the two FIR instances and the output port to the IRQ\_F2P port of the processing\_system7\_0 instance.

1. Double-click on the *processing\_system7\_0* instance to open the re-customization form.

2. Select the *Interrupt* entry in the left pane, click on the *Fabric Interrupts* check box in the right.

3. Expand the **Fabric Interrupts > PL-PS Interrupt Ports > IRQ\_F2P** entry in the right, and click the check-box of *IRQ\_F2P[15:0]*.

4. Click **OK**.

5. Add an instance of the *concat* IP.

6. Connect the interrupt port of each of the FIR instances to the two input ports of the *xlconcat\_0* instance.

7. Connect the output port of the *xlconcat\_0* instance to the **IRQ\_F2P** port of the
    *processing\_system7\_0* instance.

	At this stage the design should look like shown below (you may have to click the regenerate button).

    <p align="center">
    <img src ="./images/lab4/Figure13.png">
    </p>
    <p align = "center">
    <i>Generated design after IRQ_F2P interface enabled</i>
    </p>

#### Verify addresses and validate the design. Generate the system\_wrapper file, and add the provided Xilinx Design Constraints (XDC).

1. Click on the **Address Editor**, and expand the **processing\_system7\_0 > Data** if necessary.

	The generated address map should look like as shown below.

    <p align="center">
    <img src ="./images/lab4/Figure14.png">
    </p>
    <p align = "center">
    <i>Generated address map</i>
    </p>

2. Run *Design Validation* (**Tools > Validate Design**) and verify there are no errors
3. In the sources view, right-click on the block diagram file, **system.bd**, and select **Create HDL Wrapper** to update the HDL wrapper file. When prompted, click OK with the Let Vivado manage wrapper and auto-update option.

4. Click **Add Sources** in the Flow Navigator pane, select **Add or Create Constraints**, and click
    **Next**.
5. Click the *Add Files* button, browse to the **{labs}\lab4** folder, select **pynq\_z2\_audio\_constraints.xdc**.
6. Click *Copy constraints files into project* and then click **Finish** to add the file.
7. Click on the **Generate Bitstream** in the Flow Navigator to run the synthesis, implementation, and bitstream generation processes.
8. Click **Save, Yes**, and **OK** if prompted to start the process.
9. When the bit generation is completed, a selection box will be displayed with *Open Implemented Design* option selected. Click **Cancel**.

### Export to SDK and create Application Project

#### Export the hardware along with the generated bitstream to SDK.

1. Select **File > Export > Export Hardware…**

2. Make sure that *Include Bitstream* option is selected and click **OK**, leaving the target directory set to local project directory.

3. Select **File > Launch SDK**

4. Click **OK**.

5. In SDK, select **File > New > Board Support Package**.

6. Click **Finish** with the default settings (with **standalone operating system**).

	This will open the *Software Platform Settings* form showing the OS and libraries selections.

7. Click **OK** to accept the default settings, as we want to create a *standalone\_bsp\_0* software platform project without requiring any additional libraries support.

	The library generator will run in the background and will create **xparameters.h** file in the **{labs}\lab4\audio\audio.sdk\standalone\_bsp\_0\ps7\_cortexa9\_0\include** directory.

8. Select **File > New > Application Project**.

9. Enter **TestApp** as the *Project* Name, and for Board Support Package, choose Use Existing (standalone\_bsp should be the only option)

10. Click **Next**, select **Empty Application** and click **Finish**

11. Select *TestApp* in the project view, right-click the *src* folder, and select **Import**.

12. Expand **General** category and double-click on **File System**.

13. Browse to **{labs}\lab4** folder and click **OK**.

14. Select both **pynq\_z2\_testapp.c** and **pynq\_z2\_audio.h** and click **Finish** to add the file to the project.

	The program should compile successfully.

### Verify the Design in Hardware

#### Connect a micro-usb cable between a PC and the JTAG port of the board. Connect an audio patch cable between the Line In jack and the speaker (headphone) out jack of a PC. Connect a headphone to the Line Out jack on the board. Power On the board.

1. Connect a micro-usb cable between a PC and the JTAG port of the board.

2. Connect an audio patch cable between the **Line In** jack and the **speaker out** (headphone)  jack of a PC.

3. Connect a headphone to the *HP+MIC* jack on board. Power **ON** the board.

4. Select **Xilinx > Program FPGA**.

5. Make sure that the **system\_wrapper.bit** bitstream is selected and the **BMM file** field is blank.

6. Click **Program**.

	This will configure the FPGA.

7. Double-click **corrupted\_music\_4KHz.wav** or some other wave file of interest to play it using the installed media player. Place it in the continuous play mode.

8. Right-click on the *TestApp* in the Project Explorer pane and select **Run As > Launch On Hardware (System Debugger)**.

	The program will be downloaded and run. If you want to listen to corrupted signal then set the **SW0 OFF**. To listened the filtered signal set the **SW0 ON**.

9. When done, power OFF the board.

10. Exit SDK and Vivado using **File > Exit**.

## Conclusion

In this lab, you added INTERFACE directive to create an IP-XACT adapter. You generated the IP-XACT
adapter during the implementation phase. You then created a processor system using IP Integrator,
integrated the generated IP-XACT adapter, and tested the system with the provided application.

#### Answer

1. Answer the following questions:

	Estimated clock period: **8.70 ns**    
	Worst case latency: **174**    
	Number of DSP48E used:  **3**  
	Number of BRAMs used:  **0**  
	Number of FFs used: **167**  
	Number of LUTs used: **154**  

## Appendix

### Create a Project using Vivado GUI

#### Launch Vivado and create an empty project targeting the Pynq (xc7z020clg400-1)

1. Open Vivado by selecting **Start > Xilinx Design Tools > Vivado 2018.2**

2. Click **Create New Project** to start the wizard. You will see the Create a New Vivado Project dialog box. Click **Next**.

3. Click the Browse button of the Project Location field of the New Project form, browse to **{labs}\lab4**, and click **Select**.

4. Enter **audio** in the Project Name field. Make sure that the *Create Project Subdirectory* box is checked. Click Next.
	<p align="center">
	<img src ="./images/lab4/FigureA1.png">
	</p>
	<p align = "center">
	<i>Project Name entry</i>
	</p>
5. Select *RTL Project* in the Project Type form, and click **Next**.

6. Select *Verilog* as the Target language and Simulator Language in the Add Sources form, and click **Next**.

	<p align="center">
	<img src ="./images/lab4/FigureA2.png">
	</p>
	<p align = "center">
	<i>Add sources to new project</i>
	</p>
7. Click **Next** two times to skip *Adding Existing IP* and *Add Constraints* dialog boxes

8. In the *Default Part* form, select Boards, and select pynq-z2. Click **Next**.
    <p align="center">
    <img src ="./images/lab4/FigureA3.png">
    </p>
    <p align = "center">
    <i>Boards and Parts selection</i>
    </p>
9. Check the Project Summary and click **Finish** to create an empty Vivado project.

### Creating the System Using the IP Integrator

#### Use the IP Integrator to create a new Block Design, and generate the ARM Cortex-A9 processor based hardware system.

1. In the *Flow Navigator*, click **Create Block Design** under IP Integrator
    <p align="center">
    <img src ="./images/lab4/FigureA4.png">
    </p>
    <p align = "center">
    <i> Create IP Integrator Block Diagram</i>
    </p>

2. Enter **system** for the design name and click **OK**.

3. IP from the catalog can be added in different ways. Click on *Add IP* in the message at the top of the Diagram panel, or click the Add IP icon in the block diagram side bar, press Ctrl + I, or right-click anywhere in the Diagram workspace and select Add IP.

4. Once the IP Catalog is open, type “zy” into the Search bar, find and double click on **ZYNQ7 Processing System** entry, or click on the entry and hit the Enter key to add it to the design.
	<p align="center">
	<img src ="./images/lab4/FigureA5.png">
	</p>
	<p align = "center">
	<i>The Zynq IP Block</i>
	</p>

5. Notice the message at the top of the Diagram window that Designer Assistance available. Click on **Run Block Automation** and select *processing\_system7\_0*

6. Click **OK** when prompted to run automation.
    <p align="center">
    <img src ="./images/lab4/FigureA6.png">
    </p>
    <p align = "center">
    <i>Zynq Block with DDR and Fixed IO ports</i>
    </p>  

7. In the block diagram, double click on the Zynq block to open the Customization window for the
Zynq processing system.

	A block diagram of the Zynq should now be open, showing various configurable blocks of the Processing System.

	At this stage, the designer can click on various configurable blocks (highlighted in green) and change the system configuration.

#### Configure I/O Peripherals block to use UART 1 and I2C 1 peripherals, disabling other unwanted peripherals. Uncheck Timer 0. Enable FCLK\_CLK1, the PL fabric clock and set its frequency either to 10.000 MHz for the Pynq

1. Select the MIO Configuration tab on the left to open the configuration form and expand I/O Peripheral in the right pane.

2. Click on the check box of the I2C 1 peripheral. Uncheck USB0, SD 0, ENET 0, GPIO > GPIO MIO as we don’t need them.

3. Expand the Application Processing Unit group in the Select the MIO Configuration tab and uncheck the **Timer 0**.

4. Select the **Clock Configuration** in the left pane, expand the PL Fabric Clocks entry in the right, and click the check-box of *FCLK\_CLK1*.

5. Change the Requested Frequency value of FCLK\_CLK1 to **10.000 MHz**
    <p align="center">
    <img src ="./images/lab4/FigureA7.png">
    </p>
    <p align = "center">
    <i>Enabling and setting the frequency of FCLK_CLK1</i>
    </p> 
6. Click **OK**.
	Notice that the Zynq block only shows the necessary ports.

#### Add the provided I2C-based IP

1. In the Flow Navigator pane, click **Settings** under Project Manager.
    <p align="center">
    <img src ="./images/lab4/FigureA8.png">
    </p>
    <p align = "center">
    <i>Invoking Project Settings</i>
    </p> 
    
2. Expand **IP > Repository** in the left pane.

3. Click the + button. Browse to **{labs}\lab4\ip\_repo** and click **Select**.
	The directory will be scanned and added in the IP Repositories window, and two IP entry will be detected.

4. Click **OK** to accept the settings.

#### Instantiate zed\_audio\_ctrl and GPIO with width of 2 bits on channel 1 and width of 1 bit input only on channel 2. Run connection automation to connect them.

1. Click the **Add IP** button if the IP Catalog is not open and search for AXI GPIO in the catalog
by typing *gpi* and double-click on the AXI GPIO entry to add an instance.

2. Click on the **Add IP** to Block Design button.

3. Double-click on the added instance and the Re-Customize IP GUI will be displayed.

4. Change the Channel 1 width to **2**.

5. Check the **Enable Dual Channel** box, set the width to *1* input only, and click **OK**.

6. Similarly add an instance of the **zed\_audio\_ctrl**.

7. Notice that Design assistance is available. Click on *Run Connection Automation*, and select
**axi\_gpio\_0/S\_AXI**

8. Click **OK** to connect it to the M\_AXI\_GP0 interface.
	Notice two additional blocks, Proc Sys Reset, and AXI Interconnect have automatically been added to the design.

9. Similarly, click on *Run Connection Automation*, and select **zed\_audio\_ctrl\_0/S\_AXI**.

#### Make IIC\_1, GPIO, FCLK\_CLK1, and zed\_audio\_ctrl ports external.

1. Select the GPIO interface of the *axi\_gpio\_0* instance, right-click on it and select **Make External** to
create an external port. This will create the external port named GPIO and connect it to the peripheral.

2. Select the GPIO2 interface of the *axi\_gpio\_0* instance, right-click on it and select **Make External**
to create the external port.

3. Similarly, selecting one port at a time of the *zed\_audio\_ctrl\_0* instance, make them external.

4. Similarly, make the IIC\_1 interface and FCLK\_CLK1 port of the processing\_system7\_0 instance
external.

	At this stage the design should look like shown below (you may have to click the regenerate layout button).
	<p align="center">
	<img src ="./images/lab4/FigureA9.png">
	</p>
	<p align = "center">
	<i>Block design after I2C based zed_audio_ctrl core added and connections made for the Pynq</i>
	</p> 

