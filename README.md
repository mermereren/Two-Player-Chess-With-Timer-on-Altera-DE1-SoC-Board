# Two-Player Chess With Timer on Altera DE1-SoC Board
## Introduction
This code is written for the final project of EE443 Microprocessors course (Fall 2021/2022, Bogazici University) by 

**Eren Mermer:** VGA & Graphics Implementation, Chess Algorithm, Timer, Audio Sampling and Implementation, External Inputs and Outputs (7 segment display of timer, pushbutton, switches). 

**Sait Sevban Cander:** PS/2 Mouse Implementation, Board Display Algorithm, Chess Piece Art, Project Report

**Uğur Tekinalp:** PS/2 Mouse Implementation, Chess Algorithm, Board Display Algorithm

In this project, a standard 2-player chess game with time mode is implemented on the Altera DE1-SoC board. 
The game can be played using a PS/2 mouse. The game board is displayed on the screen with 640x480 resolution. 
The timing options can be selected using the switches on the DE1-SoC board and the remaining time is shown on the seven-segment displays of the board. 

## Task Description and Design Implementation
We used different inputs and outputs for the implementation. These individual implementations are explained in the sub-parts below. 
The code was written in C. Both interrupt and polling methods are used in the code. 
The interrupt method is used for obtaining data from the mouse and the polling method is used to obtain data from the timer and the pushbuttons. 
The game does not require additional hardware that would not be available to an average computer. The game is relatively fast and the moves are displayed smoothly. 

### Implementing the Graphics
The first task of this project was to print the chess board to the screen without the pieces. 
Before starting that, some pixel calculations are done. The pixel buffer provides an image resolution of 320x240 pixels. 
Since a square-shaped chess board is implemented to the screen and it is indented to the middle, only 240x240 pixels are used. 
There are 64 squares on the chess board, so every square has 30x30 pixel resolution before replication. 
So, we defined an array that keeps the data of pixels of the chess board. This array changes its color data every 30 pixels both in horizontal or vertical. 
Then the data inside this array is printed to the screen. This resulted in a screen as shown. 
![](https://github.com/mermereren/Two-Player-Chess-With-Timer-on-Altera-DE1-SoC-Board/blob/main/images/emptyboard.jpg?raw=true)

After printing the empty board, the second task is to print the chess pieces on the screen. This means the pixel data of 30x30 chess piece images are needed. 
The images we wanted were not available on the internet, so we drew it ourselves with a free online pixel art drawing tool. 
After drawing the pieces, we transformed those drawings into 30x30 arrays. The arrays are included in the code as a separate txt file. 
In the array, the elements representing the pixels of the piece are valued as zero. After getting the pixel data of each piece, 
another function that prints these data to the screen is needed. We created another function that takes the most left and the top pixel of each square as a reference, 
then prints the desired piece with its corresponding color to the desired square of the chess board. The chess board with pieces displayed is shown below.

![](https://github.com/mermereren/Two-Player-Chess-With-Timer-on-Altera-DE1-SoC-Board/blob/main/images/boardwithpieces.jpg?raw=true)

### Using the PS/2 Mouse
To implement a mouse to this project, the PS2 port of the DE1-SoC board is used. PS/2 mice send the data in 3-byte packets. 
To get the data in the first place, the initialization command(0xF4) is written to the base address of the PS2 port. 
Then, to get interrupts from the mouse, the interrupts are enabled by sending 0x11 to the control register of the mouse. 
At the beginning of the program, interrupt services and the GIC (General Interrupt Controller) are initialized in C programming. 
Then, every time the mouse sends the data, the program goes to the interrupt subroutine and checks for the ID of the interrupt. 
The ID of the PS2 port is 79. So, if the interruption ID equals 79, the program goes to the next step which is handling the interrupt.
When the program knows that the data is incoming with an interrupt, it should decode and save the data. 
In this project, first the data is decoded by using the fact that the bit-3 of the first byte is always logic 1. 
However this information is not enough because the bit-3 of the second and third bye can also be logic 1. 
Since we are not using the middle button and the left button of the mouse, those bits will always be logic 0. So 3 bits of the first byte is known. 
This fact is used to differentiate the first byte and then to save the second and third byte respectively. 
Lastly, a cursor is shown on the screen according to the horizontal and vertical coordinates of the mouse. 
The color of the cursor is bright green in order to follow the cursor easily. 
Every time the mouse moves, the change in the coordinates of the mouse are added to the coordinates of the cursor in the interrupt subroutine. 
After adding the coordinates, the cursor is printed to the screen again. When the left button of the mouse is pressed, the program saves the present coordinates of the cursor.

### Calculating and Displaying Time and Ending the Turn
There are also time options in the project. Thus, players can choose in which time range they want to play. 
They can choose the increment of the time (in seconds) for each move and the total time (in minutes) for each player by using switches. 
To play with increments, players should turn on one of the switches from the increment part (leftmost 3 switches, bits 7 to 9 of the switch address), 
and one of the switches from the total time part. If none of the increment switches are turned on, the time mode will nove have increments
i.e. the time will not increase after making a move. 
If none of the time switches are turned on, the game will not have a time mode. 
For example, if the player wants to play in 5+3 mode, switch 2 and switch 8 should be turned on. Note that switch 0 is the rightmost switch and switch 9 is the leftmost switch.

In order to implement the timer to the program, the FPGA interval timer is used. First, the FPGA timer is configured for counting one second. 
The FPGA timer raises its interrupt flag when one second is completed. 
The program lowers the interrupt flag and decreases one second from the time of the player playing the current turn. 
The remaining time for each player is displayed on the seven-segment displays on the DE1-SoC board.
Each player should press any of the pushbuttons shown in Fig.5 after making a move. 
When the player presses a pushbutton, their counter will stop and then the remaining time of the opponent will start counting down.
	
### Algorithm of the Game
In this project, a 8x8 two dimensional integer array is used to represent the chess board. Each piece has its special number in the array. 
To store whether the piece is black or white, another 8x8 two dimensional integer array is used. 
Every time a move is made by a player with the mouse, these two arrays are changed accordingly and the pieces on the screen are drawn again. 
Also after each move, when a player presses one of the pushbuttons to end his/her turn, 
the board is flipped so that the other player does not have to play from a reversed point of view. 
To do that, the two dimensional array that contains the data of the pieces is changed into its symmetric version with respect to the origin. 
This way, both players can see the chess board from their own perspective. 
There is another 8x8 two dimensional integer array which stores the possible moves of the selected piece. 
The algorithm calculates the possible moves of the selected piece and writes the data to this array. Then the array is used to calculate if the move is valid or not. 
If the move is valid, the program performs the move and displays it on the screen. 
The game is over when one of the players runs out of time or the player’s king is directly attacked by one of the pieces of the opponent and
has no possible move to escape the check e.g. is checkmated.

### Audio
The program also uses the  audio out port of the codec. The task for this part is to play a sound effect every time that a player makes a valid move. 
Firstly, we searched for license-free sounds on the internet. After finding and downloading an appropriate sound that lasts approximately half a second,
we converted the sound data into a one dimensional array using the MATLAB function “audioread”. 
The function uses 44.1kHz sample rate, which is compatible with Codec on the DE1-SoC board. Then the array is saved in a file called ‘sfx.txt’. 
The main function of the program includes that file and sends the sound data to the audio out port when a player makes a valid move. 
To hear the sound, players should connect a speaker or a headset to the audio out port of the DE1-SoC board.

### References
“Altera University Program DE1-SoC Computer Manual.” [Online]. Available: https://ftp.intel.com/Public/Pub/fpgaup/pub/Intel_Material/18.1/Computer_Systems/DE1-SoC/DE1-SoC_Computer_ARM.pdf. [Accessed: 18-Jan-2022]. 
