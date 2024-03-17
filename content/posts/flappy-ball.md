+++
authors = ["Ignacio Sadurni"]
title = "Flappy Ball Game"
date = "2023-12-15"
description = "A brief description of Hugo Shortcodes"
tags = [
    "Fundamentals of Computing",
    "C",
    "XQuartz",
]
categories = []
series = []
+++

Designed [Flappy Bird](../../images/Flappy_Ball.jpg) game in C, incorporating player controls, obstacle generation, and scoring mechanics. Implemented XQuartz for graphics and animations, enhancing visual experience. Implemented precise collision detection to ensure accurate interaction between player and obstacles.

---

**NOTE.** For more information on how the game works, read the info below:

<img src="../../images/Flappy_Ball.jpg" alt="text">

**SOURCE CODE.** The following is written in C programming language and interpreted using gcc compiler:

{{< tabgroup align="left" style="code" >}}
{{< tab name="Makefile" >}}

```shell
project: project.c
	gcc project.c gfx2.o -lX11 -lm -o project

clean:
	rm project 
```

{{< /tab >}}
{{< tab name="Main Driver" >}}

```c
// Author: Ignacio Sadurni
// Date: 12.8.2023
// Lab 11 (Mini) Final Project
// This program is a simple game where the ball must fly between the green pipes for as long as you can, obtaining a higher score.

# include <stdio.h>
# include <stdbool.h>
# include <unistd.h>
# include <stdlib.h>
# include <stdio.h>
# include <math.h>
# include "gfx2.h"
# define width 800
# define height 600

typedef struct {
	int game_speed;
	int gravity;
	} Modes;

// FUNCTION PROTOTYPES /////////////////////////////////////////////////////////////////////////
void draw_pipe(int);
void draw_ball(int,int);
bool check_bounds(int,int,int);
void disp_menu(bool, int, int);
void disp_rules();

// MAIN DRIVER /////////////////////////////////////////////////////////////////////////////////

int main() {

	// declare variables
	Modes settings[2]; // use struct for difficulty settings
	settings[0].game_speed = 50000; // game speed for easy mode
	settings[1].game_speed = 25000; // game speed for hard mode
	settings[0].gravity = 10;
	settings[1].gravity = 20;
	bool first = true;
	int score = 0;
	int high_score = 0;
	int mode = 1;
	gfx_open(width, height, "Mini Project"); // open tab
	gfx_clear_color(102,204,255); // background color to blue

	while(1) { // Runs until user quits app by pressing q in the menu

		// declare variables that reset to original value:
		bool over = false; // flag for game while it is not over
		bool down = true; // flag to indicate direction of ball
		int x_pipe = width; // position of pipe (x)
		int x = 200; // position of ball (x)
		int y = height/4; // position of ball (y)
		int count = 0; // counter

		if(high_score < score) high_score = score; // updates high score
		disp_menu(first, high_score, mode); // call function to display menu

		char input = gfx_wait(); // get user input

		if(input == 'r') {
			disp_rules(); // if user presses r call function to display rules
			over = true;
		}
		else if(input == 'q') break; // quit app
		else if(input == 'd') { // change difficulty
			if(mode == 1) mode = 2;
			else if(mode == 2) mode = 1;
			over = true;
		}

		score = 0; // set score to 0

		while(!over) { // Loop runs the actual game
			if(mode == 1) usleep(settings[0].game_speed); // create delay for easy mode
			else if(mode == 2) usleep(settings[1].game_speed); // create delay for hard mode
			input = '0'; // reset user input
			gfx_clear(); // clear screen
			draw_ball(x,y); // call function to draw ball
			draw_pipe(x_pipe); // call function to draw pipe
			gfx_flush();

			if(gfx_event_waiting()) { // if user presses key in game
				input = gfx_wait();
			}

			if(abs(x-x_pipe) < 60) { // check if ball is within bounds of pipe
				over = check_bounds(x,y,x_pipe); // calls function to check height of ball
			}

			if(input == ' ') { // is user presses space key
				down = false; // set down to false, indicating that the ball must go up
			}

			x_pipe -= 20; // move pipe to the left
			score++; // increase score

			if(x_pipe <= 0) { // if pipe reaches the left of the screen
				x_pipe = width; // change pipe position to the right side of the screen
			}

			switch(down) {
				case true: // if ball is going down, increase y
					if(mode == 1) y+=settings[0].gravity; // easy mode gravity
					else y+=settings[1].gravity; // hard mode gravity
					break;
				case false: // if ball is going up, decrease y
					if(mode == 1) y-=settings[0].gravity;
					else y-=settings[1].gravity-5;
					count++; // update counter
				break;
			}	

			if(count == 10) { // when counter is 10, change direction of ball to go down again
				down = true;
				count = 0;
			}
			first = false; // indicates it is not the first iteration
		}
	}
	return 0;
}

// FUNCTIONS ///////////////////////////////////////////////////////////////////////////////////

void draw_pipe(int x) { // This function draws both green pipes
	gfx_color(0,255,0);
	gfx_fill_rectangle(x, 0, 50, 200);
	gfx_fill_rectangle(x-20, 200, 90, 25);
	gfx_fill_rectangle(x, 400, 50, 475);
	gfx_fill_rectangle(x-20, 375, 90, 25);
	gfx_color(0,0,0);
	gfx_rectangle(x, 0, 50, 200);
	gfx_rectangle(x-20, 200, 90, 25);
	gfx_rectangle(x, 400, 50, 475);
	gfx_rectangle(x-20, 375, 90, 25);
}

void draw_ball(int x, int y) { // This function draws the yellow ball
	gfx_color(255,255,26);
	gfx_fill_circle(x,y,20);
	gfx_color(0,0,0);
	gfx_circle(x,y,20);
}

bool check_bounds(int x, int y, int x_pipe) { // This function check the bounds to see if the ball touched the pipe
	if(y-20<=225 || y+20>=375) return true;
}

void disp_menu(bool first, int score, int mode) { // This function displays the menu
	gfx_clear();
	gfx_color(0,0,0);
	gfx_text(310,200,"WELCOME TO FLAPPY BALL!!!");
	gfx_text(310,300,"Press any key to continue");
	gfx_text(310,320,"Press r to view the rules");
	gfx_text(310,340,"Press q to quit the game!");
	gfx_text(310,360,"Press d to alter gamemode");
	draw_ball(330,250);
	draw_ball(620,300);
	draw_pipe(100);
	draw_pipe(625);
	if(first == false) { // After the first game the score is displayed in the menu
		char scoreChar[20];
		sprintf(scoreChar, "High Score: %d", score); // Converts score (int) to a string
		gfx_text(335,30,scoreChar); // displays high score
	}
	if(mode == 1) gfx_text(355,50,"Easy Mode"); // display difficulty / gamemode
	else if(mode == 2) gfx_text(355,50,"Hard Mode");
	gfx_flush();
}

void disp_rules() { // This function displays the rules
	gfx_clear();
	gfx_color(0,0,0);
	gfx_text(270,200,"The yellow ball will descend over time");
	gfx_text(270,250,"Press space to jump or 'flap'  upwards");
	gfx_text(270,300,"You cannot hit the moving green pipes!");
	gfx_text(270,400,"Helpful tip: do not spam the space key");
	gfx_text(270,450,"Press any key to continue, have fun!!!");
	draw_ball(90,300);
	draw_pipe(100);
	draw_pipe(625);
	gfx_wait(); 
}
```

{{< /tab >}}
{{< tab name="GFX" >}}

```c
// gfx2.h

#ifndef GFX2_H
#define GFX2_H

#include <X11/Xlib.h>
#include <X11/cursorfont.h> 

// Open a new graphics window. 
void gfx_open( int width, int height, const char *title );

// Flush all previous output to the window. 
void gfx_flush();

// Change the current drawing color. 
void gfx_color( int red, int green, int blue );

// Clear the graphics window to the background color. 
void gfx_clear();

// Change the current background color. 
void gfx_clear_color( int red, int green, int blue );

// Check to see if an event is waiting. 
int gfx_event_waiting();

// Wait for the user to press a key or mouse button. 
char gfx_wait();

// Return the X and Y coordinates of the last event. 
int gfx_xpos();
int gfx_ypos();

// Return the X and Y dimensions of the screen (monitor). 
int gfx_xsize();
int gfx_ysize();

// Draw a point at (x,y) 
void gfx_point( int x, int y );

// Draw a line from (x1,y1) to (x2,y2) 
void gfx_line( int x1, int y1, int x2, int y2 );

// Draw a circle centered at (xc,yc) with radius r 
void gfx_circle( int xc, int yc, int r );

// Display a string at (x,y) 
void gfx_text( int x, int y , const char *text );

// -----------------------------
// new functions added 10/4/23:
// -----------------------------

// Draw a filled circle centered at (xc,yc) with radius r 
void gfx_fill_circle( int xc, int yc, int r );

// Draw an ellipse centered at (xc,yc) with radii r1 and r2 
void gfx_ellipse( int xc, int yc, int r1, int r2 );
 
// Draw an arc whose top left corner of its bounding rectangle is at (x,y),
//  with width w and height h, starting at angle a1 and sweeping an angle of a2 (degrees);
//  a1 of 0 is at the 3 O'Clock position, and the a2 sweep is positive counter-clockwise 
void gfx_arc( int x, int y, int w, int h, int a1, int a2 );

// Draw a filled arc (similar description to gfx_arc)
void gfx_fill_arc( int x, int y, int w, int h, int a1, int a2 );

// Draw a rectangle with top-left corner at (x,y) with width w and height h 
void gfx_rectangle( int x, int y, int w, int h );

// Draw a filled rectangle (similar description to gfx_rectangle)
void gfx_fill_rectangle( int x, int y, int w, int h );

// Draw a polygon whose "num_pts" number of corners are in the "pointsarr" array
//  (note: uses Xlib's XPoint struct) 
void gfx_polygon( XPoint *pointsarr, int num_pts );

// Draw a filled polygon (similar description to gfx_polygon)
void gfx_fill_polygon( XPoint *pointsarr, int num_pts );

// Change the font that gfx_text will use 
//   (see the file /usr/share/X11/fonts/misc/fonts.alias for possible fonts)
void gfx_changefont( char * );

// change the cursor (mouse pointer)
//   (see the file /usr/include/X11/cursorfont.h for possible cursors)
void gfx_changecursor( int );

#endif
```

{{< /tab >}}
{{< /tabgroup >}}

{{< notice warning >}}

The GFX2 Library is not included in the tabs above.

{{< /notice >}}