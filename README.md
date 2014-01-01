EE201 Text Editor
=================

Text editor created for my EE 201 - Introduction to Digital Circuits final project with Aidan Blant. A video demonstration will be uploaded soon.

###Project Framework
Our main goal for this project was to implement a text editor using our Nexys-3 that allows a user to use an attached keyboard to type and display text on screen. The text editor has the following capabilities: displaying the alphabet, displaying a blinking cursor, changing text color, changing text size, scrolling text, and deleting text. Due to time constraint and physical constraints of the Nexys-3 itself, we were unable to implement a function for every key on the keyboard (no numbers 0-9, no Ctrl, Shift, CapsLock, Tab, Alt, extra punctuation functionality); however, the program certainly works for very simple text editing.

The Xilinx project is broken up into 7 modules. The PS2_Controller (and its two sub modules Altera_UP_PS2_Data_In and Altera_UP_PS2_Command_Out) interpret the 11-bit words provided by the PS2 interface and provide a simpler way of dealing with keyboard input by providing key codes when buttons are pressed. We built the text_editor_RAM module; it serves as a synchronous-write, asynchronous-read 512 byte (512 locations x 8 bit) RAM to store the current text document. Because every character is represented by an 8-bit key code in the PS2 interface, this means we can store 512 characters. The hvsync_generator module was provided on Blackboard and was used for VGA signal synchronization for output to a VGA monitor. The module text_editor_keyboard_controller handles some logic to filter out unwanted key codes from the PS2_Controller module, and only sends new 8-bit words to the top module when a key is released (rather than just pressed/held). The top module handles all logic for interpreting keyboard input, storing the correct key codes to RAM, and drawing each character on the screen.

###Technical difficulties
1. Finding a way to input keyboard presses and store them as characters
2. Finding a format to store characters that allowed us to add, edit, or delete contents
3. Displaying characters in memory to display output through VGA
4. Memory management

###Solutions
To create a text editor, we first focused our efforts into ensuring we could receive keyboard input and store these inputs as characters on the FPGA.  We acquainted ourselves with the use of the ps/2 module which allowed us the use of a USB keyboard. We were then able to use the commands the keyboard translated to us as 2-digit hexadecimal code to save as character data into our memory. A problem that was never fixed is that sometimes the keyboard does not seem to initialize properly, and will not send data to the FPGA. We attempted to force it to reset by sending the Reset command (0xFF) in the keyboard controller module on Reset, but that did not seem to solve the problem. The best method to ensure the keyboard will work is to have it plugged in before turning on the FPGA, turning the FPGA on, programming the bit file, and then removing and re-plugging the keyboard back in. This usually works, although it is important to note that not all keyboards will work (they must draw 100mA or less power from the USB port).

To draw characters to the display, we represented each character as a 256-bit (16x16 pixel) array. This display information is essentially a picture, which we constructed bit by bit. By defining these as parameters, Xilinx knew to store these in ROM rather than using precious flip flops on the FPGA.

Storage consisted of the largest size RAM we could implement without sacrificing any of the other necessary features of the program.  We attempted a 1024 byte RAM initially, but this used over 100% of the board’s resources during synthesis, so we were forced to move to a smaller size. Instead, we used a 512 location 8-bit RAM to store the key code of characters in each location.  The default value for these locations was the ‘space’ character (0x29), conveying no output information.  When typing, a user is actually replacing these space values with a character.  Deletion worked in much the same manner, merely replacing the existing character with a ‘space’ character.

Display of letters utilized an algorithm which used the ‘currentX’ and ‘currentY’ as the output wave scanned for output to determine a) which row the current pixel is in b) which column the current pixel was in and c) using these to determine what location of memory to gather output information from the RAM. After determining which character in RAM should be displayed, we next had to determine which pixel of the 256-bit character parameter we should be displaying. It took us a while to figure out the best approach and we thought we were moving along smoothly; however, we ran into a problem that many groups had: Xilinx only allows use of the % (modulus) and / (divide) operators when modding or diving by a power of 2. This threw a wrench in our plans, so we had to rework some of our math to make sure those operations always worked.

###Implementation Details
The current implementation uses a virtual 512 byte RAM located on the FPGA itself; this used up almost all of the available slices available to us on the board. We would have liked to instead been able to use one of the actual RAMs on the Nexys-3, so that we could have stored far more characters and possibly even saved out the characters to a true .txt file to transfer it to a computer. Unfortunately, we were unable to devise a way to both write to and read from one of the external RAMs quickly enough for simultaneous VGA output and keyboard input. 

###Instructions
#####Normal Typing:
+ Letters **A-Z**: type a capital letter
+ **Space**: inserts a space at the cursor
+ Comma (**,**) Period (**.**) and Apostrophe (**‘**): work normally
+ 1 (one) inserts an exclamation point (**!**)
+ **Insert**: Places a solid, filled-in block at the cursor (for pixel art)
+ **Backspace**: delete previous letter and move cursor back one
+ **Delete**: delete character at cursor without moving cursor

#####Keyboard commands:
+ **Arrow Keys**: Move cursor up/down/left/right
+ **+** & **-** keys (On Keypad ONLY): Increase/decrease text size
+ **Page Down** & **Page Up**: Scroll text up and down
+ **F1**: Toggle Red color of text
+ **F2**: Toggle Green color of text (Will default to green if all are off)
+ **F3**: Toggle Blue color of text
