# VGA_Controller
<details>
<summary><h3>PROJECT DETAILS</h3></summary>
 <br>
Creating a VGA (Video Graphics Array) controller using Xilinx Vivado involves generating VGA signals to control a display and render graphics on a screen. The VGA controller requires handling timing signals for horizontal and vertical sync, which dictate when and where pixels are drawn on the screen.

Here’s how you can implement a basic VGA controller in Xilinx Vivado:

### Steps to Create a VGA Controller

#### 1. **Understand VGA Timing**
   A VGA controller needs to generate three main signals:
   - **Horizontal sync (HSYNC)**: Signals the start of a new row.
   - **Vertical sync (VSYNC)**: Signals the start of a new frame.
   - **RGB (Red, Green, Blue)**: Determines the color of each pixel on the screen.

   Standard VGA resolution (640x480 @ 60Hz) requires precise timing:
   - **Pixel clock**: 25.175 MHz (for 640x480 resolution)
   - **Horizontal timing**:
     - Visible area: 640 pixels
     - Front porch: 16 pixels
     - Sync pulse: 96 pixels
     - Back porch: 48 pixels
     - Total: 800 pixels
   - **Vertical timing**:
     - Visible area: 480 lines
     - Front porch: 10 lines
     - Sync pulse: 2 lines
     - Back porch: 33 lines
     - Total: 525 lines

#### 2. **Create a Verilog/VHDL Module**

Start by writing a Verilog or VHDL module to generate the correct timing signals for HSYNC and VSYNC based on the timing parameters above.



```verilog
module vga_rgb_color(
    input wire clk,             // Pixel clock (25.175 MHz for 640x480 VGA)
    input wire reset,           // Reset signal
    output reg hsync,           // Horizontal sync signal
    output reg vsync,           // Vertical sync signal
    output reg [2:0] red,       // Red signal (3 bits)
    output reg [2:0] green,     // Green signal (3 bits)
    output reg [2:0] blue,      // Blue signal (3 bits)
    output reg display_enable   // High when pixel_x and pixel_y are within visible area
);

    // VGA 640x480 @ 60Hz timing parameters
    parameter H_DISPLAY    = 640;  // Horizontal visible area
    parameter H_FRONT      = 16;   // Horizontal front porch
    parameter H_SYNC       = 96;   // Horizontal sync pulse
    parameter H_BACK       = 48;   // Horizontal back porch
    parameter H_TOTAL      = 800;  // Total horizontal period

    parameter V_DISPLAY    = 480;  // Vertical visible area
    parameter V_FRONT      = 10;   // Vertical front porch
    parameter V_SYNC       = 2;    // Vertical sync pulse
    parameter V_BACK       = 33;   // Vertical back porch
    parameter V_TOTAL      = 525;  // Total vertical period

    reg [9:0] h_counter = 0;  // Horizontal counter (0 to H_TOTAL-1)
    reg [9:0] v_counter = 0;  // Vertical counter (0 to V_TOTAL-1)

    // Horizontal sync and counter logic
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            h_counter <= 0;
        end else begin
            if (h_counter < H_TOTAL - 1)
                h_counter <= h_counter + 1;
            else
                h_counter <= 0;
        end
    end

    // Vertical sync and counter logic
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            v_counter <= 0;
        end else begin
            if (h_counter == H_TOTAL - 1) begin
                if (v_counter < V_TOTAL - 1)
                    v_counter <= v_counter + 1;
                else
                    v_counter <= 0;
            end
        end
    end

    // Horizontal sync signal generation
    always @(*) begin
        if (h_counter >= (H_DISPLAY + H_FRONT) && h_counter < (H_DISPLAY + H_FRONT + H_SYNC))
            hsync = 0; // Active low
        else
            hsync = 1;
    end

    // Vertical sync signal generation
    always @(*) begin
        if (v_counter >= (V_DISPLAY + V_FRONT) && v_counter < (V_DISPLAY + V_FRONT + V_SYNC))
            vsync = 0; // Active low
        else
            vsync = 1;
    end

    // Generate display enable signal (active when pixel within visible area)
    always @(*) begin
        display_enable = (h_counter < H_DISPLAY) && (v_counter < V_DISPLAY);
    end

    // RGB color output logic
    always @(posedge clk) begin
        if (display_enable) begin
            // Set specific RGB colors based on pixel position
            if (h_counter < 213)
                {red, green, blue} <= 9'b111_000_000; // Red color
            else if (h_counter < 426)
                {red, green, blue} <= 9'b000_111_000; // Green color
            else
                {red, green, blue} <= 9'b000_000_111; // Blue color
        end else begin
            {red, green, blue} <= 9'b000_000_000; // Black when outside visible area
        end
    end

endmodule

```
![Screenshot (10)](https://github.com/user-attachments/assets/12a2e9a2-4298-4787-8de4-a9e33f2c9b33)
![Screenshot (11)](https://github.com/user-attachments/assets/4a624d46-d92f-4e38-93fc-1c012f92c382)
![Screenshot (12)](https://github.com/user-attachments/assets/a4f669a3-9540-4397-aa87-5f181f4c4aa6)

#### 3. **Generate Pixel Data**

Once you have the timing signals, you can generate pixel data. You can create basic patterns like:
- Color bars
- Gradient
- Moving shapes

Example (solid color display):

```verilog
always @(posedge clk) begin
    if (display_enable) begin
        // Display a simple pattern (color bars)
        if (pixel_x < 213)
            {red, green, blue} <= 3'b100; // Red
        else if (pixel_x < 426)
            {red, green, blue} <= 3'b010; // Green
        else
            {red, green, blue} <= 3'b001; // Blue
    end else begin
        {red, green, blue} <= 3'b000; // Black (outside visible area)
    end
end
```

#### 4. **Clock Generation (Optional)**
You’ll need a 25.175 MHz clock for 640x480 VGA, which is lower than typical FPGA clock frequencies (e.g., 100 MHz). You can use a clock divider or Xilinx’s **Clocking Wizard** IP to generate the required frequency.

#### 5. **Simulate the Design**
Once your Verilog/VHDL code is ready, simulate it in Vivado to verify the correct generation of HSYNC, VSYNC, and pixel data.

#### 6. **Synthesize and Implement on FPGA**
After simulation:
1. Synthesize the design in Xilinx Vivado.
2. Implement the design and generate a bitstream.
3. Load the design onto the FPGA and connect it to a VGA monitor to test the display.

#### 7. **Test and Debug**
Ensure that the screen is displaying the expected patterns or colors. You may use an oscilloscope to verify the sync signals (HSYNC and VSYNC).
</details>

<details>
<summary><h3> TESTBENCH </h3></h3></summary>
 <br>
  
## TESTBENCH:
Below is a simple Verilog testbench for the VGA RGB color controller described earlier. This testbench will simulate the behavior of the VGA controller and display different RGB colors based on the pixel positions.

### Testbench for VGA RGB Color Controller

```verilog
module tb_vga_rgb_color();

    // Testbench clock and reset
    reg clk;
    reg reset;
    
    // Outputs from the VGA controller
    wire hsync;
    wire vsync;
    wire [2:0] red;
    wire [2:0] green;
    wire [2:0] blue;
    wire display_enable;

    // Instantiate the VGA controller module
    vga_rgb_color uut (
        .clk(clk),
        .reset(reset),
        .hsync(hsync),
        .vsync(vsync),
        .red(red),
        .green(green),
        .blue(blue),
        .display_enable(display_enable)
    );

    // Clock generation: 25.175 MHz
    initial begin
        clk = 0;
        forever #19.84 clk = ~clk;  // Toggle every 19.84 ns (approximate for 25.175 MHz)
    end

    // Testbench sequence
    initial begin
        // Initial reset
        reset = 1;
        #100;
        reset = 0;  // Release reset after 100 ns
        
        // Let the simulation run for a certain time
        #1000000;  // Simulate for 1 ms (adjust time as needed)

        $stop;  // Stop simulation
    end

    // Optional monitoring of VGA signals
    initial begin
        $monitor("Time: %0t | HSYNC: %b | VSYNC: %b | RED: %b | GREEN: %b | BLUE: %b | Display Enable: %b", 
                  $time, hsync, vsync, red, green, blue, display_enable);
    end

endmodule
```

### Explanation of Testbench:
1. **Clock Generation**:
   - A clock signal is generated with a 25.175 MHz frequency, which is typical for a 640x480 VGA display. The clock toggles every 19.84 ns to approximate this frequency.

![Screenshot (13)](https://github.com/user-attachments/assets/d1ec84a1-d1e6-4ae0-a9ee-55c0313f5bf9)

![Screenshot (14)](https://github.com/user-attachments/assets/c61395f4-7dff-4168-a67d-59a821e06039)

2. **Reset Sequence**:
   - The reset signal is asserted (`reset = 1`) for the first 100 ns of the simulation, and then deasserted (`reset = 0`) to allow the VGA controller to start running.

3. **Simulation Time**:
   - The simulation is allowed to run for a certain amount of time (in this case, 1 millisecond, or 1,000,000 nanoseconds). You can adjust this time as needed.

4. **Monitor Statement**:
   - The `$monitor` statement prints the values of the key signals (HSYNC, VSYNC, RGB colors, and display enable) at each time step. This helps you observe the behavior of the VGA controller in the simulation.

5. **Clock Frequency**:
   - The clock toggles approximately every 19.84 ns, giving a frequency of around 25.175 MHz, which is typical for driving a VGA display at 640x480 resolution.

### How to Run the Simulation in Vivado:
1. **Create a New Simulation Source**:
   - In Vivado, add the testbench as a new simulation source.
   - Link the VGA controller module (UUT) with the testbench.

2. **Simulate**:
   - Run the simulation for a few milliseconds to observe the RGB output signals (`red`, `green`, `blue`), and synchronization signals (`hsync`, `vsync`).
   - Check the waveform to verify the timing and signal transitions.

3. **Examine Waveforms**:
   - Use the simulation waveform viewer to check the signal behavior and make sure that the red, green, and blue signals change as expected depending on the pixel position.

By running this testbench in Vivado, you can verify the functionality of your VGA controller design. You should see the RGB color signals toggling at the expected times and the sync signals (`hsync` and `vsync`) behaving correctly.
</details>

<details>
<summary><h3> FINAL SIMULATION </h3></summary>
 <br>
  
## FINAL SIMULATION WINDOW:


### Key Signals in the Waveform
1. **`clk` (Clock Signal)**:
   - The `clk` signal is driving the entire system. It should be a regular square wave oscillating at a frequency close to 25.175 MHz (or whatever frequency you set in the testbench). 
   - Every signal in the design is synchronized to the positive edge of this clock.

2. **`reset`**:
   - Initially, the `reset` signal is high (`1`), which forces all counters and the controller to be in a reset state.
   - After a short time (in this case, 100 ns), `reset` goes low (`0`), allowing the VGA controller to start functioning.
   - While `reset` is high, there shouldn't be any significant activity on the horizontal and vertical counters, or the RGB outputs.

3. **`hsync` (Horizontal Sync)**:
   - This signal controls the horizontal synchronization of the VGA signal.
   - You should see `hsync` going low (active low) for a specific duration (96 clock cycles, according to the code). 
   - The rest of the time it should stay high, representing the front porch, active display, and back porch periods.
   
4. **`vsync` (Vertical Sync)**:
   - `vsync` controls the vertical synchronization of the VGA signal.
   - It behaves similarly to `hsync`, but it has a much lower frequency since it represents the vertical synchronization pulses (frames). It stays high for most of the frame except for the vertical sync period.
   
5. **`h_counter` and `v_counter` (Horizontal and Vertical Counters)**:
   - The `h_counter` increments every clock cycle and resets after reaching the total horizontal period (`H_TOTAL = 800`).
   - The `v_counter` increments every time `h_counter` reaches its maximum value (once per line). After reaching `V_TOTAL = 525`, it resets and begins a new frame.
   - Together, these counters define the current pixel location on the screen.

6. **`red`, `green`, `blue` (RGB Signals)**:
   - These signals represent the 3-bit RGB color outputs.
   - During the active display area (when `display_enable` is high), the `red`, `green`, and `blue` signals will vary based on the horizontal position (`h_counter`):
     - For the first section of the display, `red` is high.
     - For the next section, `green` is high.
     - For the final section, `blue` is high.
   - Outside the active display area (when `display_enable` is low), all RGB signals will be low (`0`), representing a black screen.

7. **`display_enable`**:
   - This signal is high when the current pixel is within the visible area (active display region) of the screen.
   - It goes low during the horizontal and vertical blanking intervals, where no display information is shown (i.e., the sync pulses, front porch, and back porch periods).
   - When `display_enable` is high, the RGB signals should show the respective colors for that pixel.

### What to Look for in the Waveform
- **HSYNC and VSYNC Timing**: Ensure that the `hsync` signal has the correct timing (active low for 96 clock cycles) and that the `vsync` signal behaves similarly but at a much lower frequency.
- **RGB Signals in Active Display Area**: Verify that the `red`, `green`, and `blue` signals toggle according to the expected behavior in the active display region (when `display_enable` is high).
- **Display Enable**: Make sure that `display_enable` is high during the active display region (visible area), which should correspond to the part of the waveform where valid RGB data is being output.
- **Pixel Pattern**: You should see distinct color changes based on the horizontal position (`h_counter`), as defined by the conditions in the code.

### Interpreting Colors in the Waveform
- **Red**: In the waveform, you'll see the `red` signal is high (e.g., `red = 111`) for the first section of the visible horizontal line (the first third of the display).
- **Green**: In the next section, the `green` signal goes high (`green = 111`), representing the middle third of the display.
- **Blue**: For the final third of the horizontal line, the `blue` signal should be high (`blue = 111`).
- **Black (Outside Visible Area)**: When `display_enable` is low, all RGB signals should be low (`000`), meaning the screen is black during the blanking intervals.

### Example of Expected Waveform Behavior
1. **During the Active Display Region** (when `display_enable` is high):
   - **First Third**: `red` signal will be high (`red = 111`), `green` and `blue` will be low.
   - **Middle Third**: `green` signal will be high (`green = 111`), `red` and `blue` will be low.
   - **Final Third**: `blue` signal will be high (`blue = 111`), `red` and `green` will be low.
2. **During the Blanking Interval**:
   - All RGB signals will be low (`red = green = blue = 000`).

### Conclusion
The waveform should give you a clear representation of how the VGA signals (`hsync`, `vsync`, and RGB) are generated. You should observe that:
- The `hsync` and `vsync` signals toggle at the correct intervals.
- The RGB signals output the correct color data based on the pixel location (red, green, or blue).
- The `display_enable` signal is high only during the visible portion of the screen, and the RGB signals are black outside this region.

![Screenshot (15)](https://github.com/user-attachments/assets/6b10d602-59af-41af-94fb-3a77604b309d)

