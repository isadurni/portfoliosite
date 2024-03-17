+++
authors = ["Ignacio Sadurni"]
title = "Crossword Puzzle Generator"
date = "2023-11-01"
description = ""
tags = [
	"Fundamentals of Computing",
	"C",
	]
categories = []
series = ["Theme Demo"]
+++

Developed a Crossword Puzzle Generator in C, allowing user word inputs for puzzle creation Implemented functionality to generate puzzles, solutions, and scrambled clues. Streamlined solving experience for users with seamless puzzle generation and solution presentation.

---

**NOTE.** For more information on how the program works, read the provided text and images below:

{{< notice info >}}
**How does it work?**
1. The program first prompts the user to enter a list of appropriate words. This can be done by entering them in the command line, or passing them in through a .txt file as a command line argument. The list is terminated by a single period.

2. Then the program generates a crossword puzzle as well as its solution and list of scrambled clues. The algorithm fits as many words as possible. These are outputted to the command line and can be saved in a file.

{{< /notice >}}

**COMMAND LINE INPUT.** The following is a sample of what the input process looks like:

{{< highlight html >}}

Anagram Crossword Puzzle Generator
----------------------------------
Enter a list of words:
ignacio
computer
science
engineering
notredame
puertorico
python
coding
github
programming
.
Note: 2 word(s) do not fit on the board.

{{< /highlight >}}

{{< notice error >}}
The algorithm correctly chooses to ignore any invalid words or those which do not fit in the board.
{{< /notice >}}

**COMMAND LINE OUTPUT.** The following is a sample of what the ouput in command line looks like:

{{< highlight html >}}

Crossword Puzzle:       Solution:               Clues:
-----------------       -----------------       2,   2  Across  IONGDC
|######### #####|       |.........P.....|       2,   7  Across  ENGGEINRNIE
|## ###### #####|       |..S......Y.....|       2,  10  Across  RUTMPCOE
|##      # #####|       |..CODING.T.....|       4,   4  Across  IUOROTPREC
|## ###### #####|       |..I......H.....|       4,  13  Across  ONEDAEMRT
|## #          #|       |..E.PUERTORICO.|       2,   1  Down    NCIESCE
|## # #### #####|       |..N.R....N.....|       4,   4  Down    MNAIRRGMGPO
|## # ##########|       |..C.O..........|       9,   0  Down    NTOHPY
|##           ##|       |..ENGINEERING..|
|#### ##########|       |....R..........|
|#### ##########|       |....A..........|
|##        #####|       |..COMPUTER.....|
|#### ##########|       |....M..........|
|#### ##########|       |....I..........|
|####         ##|       |....NOTREDAME..|
|#### ##########|       |....G..........|
-----------------       -----------------

{{< /highlight >}}

**SOURCE CODE.** The following is written in C programming language and interpreted using gcc compiler:

{{< tabgroup align="left" style="code" >}}
{{< tab name="Makefile" >}}

```shell
CMP = gcc
MAIN = crossword
FUNC = crosswordfunc
EXEC = runcrossword

$(EXEC): $(FUNC).o $(MAIN).o 
	$(CMP) $(FUNC).o $(MAIN).o -lm -o $(EXEC)

$(FUNC).o: $(FUNC).c $(FUNC).h 
	$(CMP) -c $(FUNC).c -o $(FUNC).o 

$(MAIN).o: $(MAIN).c $(FUNC).h
	$(CMP) -c $(MAIN).c -o $(MAIN).o 

clean:
	rm *.o
	rm $(EXEC)
```

{{< /tab >}}
{{< tab name="Main Driver" >}}

```c
// Author: Ignacio Sadurni
// Date: 10/31/2023
// Subject: Lab 8
// Program: Crossword Puzzle (Main Driver)
// This program asks the user for an input of max 20 words and it generates a crossword puzzle.

# include <stdio.h>
# include <stdlib.h>
# include <string.h>
# include "crosswordfunc.h"

int main(int argc, char *argv[]) // count number of arguments
{
	// initialize variables
	char board[size][size];
	char old_words[20][size];
	char words[20][size];
	int word_count;
	create_board(board); // call function to create game board
	if(argc == 1) { // mode 1
		word_count = get_words(old_words); // call function to get words and filter them in an array
		to_caps(old_words, word_count); // call function to change letters to caps
		sort_words(old_words, words, word_count); // call function to sort words in order of length
		int index_direction[word_count][3]; // initialize index and direction array
		for(int i=0; i<word_count; i++) { // set values to 0
			for(int j=0; j<3; j++) {
				index_direction[i][j] = 0;
			}
		}
		word_to_board(words, board,  word_count, index_direction); // call function to put first word in board
		all_words(word_count, words, index_direction, board); // call function to put all words in board
		display_board(board); // call function to display board
		clues(words, word_count, index_direction); // call function to display clues
	}
	else if(argc == 2 || argc == 3) { // mode 2 and 3
		FILE *fp = fopen(argv[2], "w"); // open file for writing for mode 3
		word_count = read_file(argv, old_words); // call function to read file for words
		to_caps(old_words, word_count); // call function to turn words to caps
		sort_words(old_words, words, word_count); // call function to sort words
		int index_direction[word_count][3]; // initialize index and direction array
		for(int i=0; i<word_count; i++) { // set values to 0
			for(int j=0; j<3; j++) {
				index_direction[i][j] = 0;
			}
		}
		word_to_board(words, board, word_count, index_direction); // call function to put first word in board
		all_words(word_count, words, index_direction, board); // call function to put all words in board
		display_board(board); // call function to display board
		clues(words, word_count, index_direction); // call function to display clues
		if(argc == 3) { // for mode 3...
			display_board2(board, fp); // call function to display board in output file
			clues2(words, word_count, index_direction, fp); // call function to display clues in output file
			system("clear"); // to clear command window
		}
	}
	else { // display error for invalid inputs
		printf("Error: Invalid Amount of Inputs\n");
	}
	return 0;
}
```

{{< /tab >}}
{{< tab name="Functions" >}}

```c
// Author: Ignacio Sadurni
// Date: 10/31/2023
// Subject: Lab 8
// Program: Crossword (Functions File)
// This is the functions file for the crossword puzzle.

# include "crosswordfunc.h"

int get_words(char words[][size]) { // FUNCTION TO GET USER INPUT (WORDS)
	printf("\nAnagram Crossword Puzzle Generator\n");
	printf("----------------------------------\n");
	int word_count = 0;
	bool subtract = false; // flag to subtract off total word count
	printf("Enter a list of words:\n");
	while(1) { // loop until user enters 20 words or period
		scanf("%s", words[word_count]); // read in the word into the array
		getc(stdin);
		if(strlen(words[word_count]) < 2 && words[word_count][0] != '.') { // check if word is 1 character long
			printf("Error: Word Ignored (Error: Word Ignored (Contains less than 2 characters)\n"); // display error message
			words[word_count][0] = '\0'; // nullify character
			subtract = true; // flag true to subtract off total word count
		}
		for(int i=0; i<strlen(words[word_count]); i++) { // cycle through letters of word
			if(isalpha(words[word_count][i]) == 0 && words[word_count][0] != '\0' && words[word_count][0] != '.') { // check if words contains non alphabetical character
				printf("Error: Word Ignored (Contains non alphabetical character(s))\n"); // display error message
				subtract = true; // flag true to subtract off total word count
				for(int j=0; j<size; j++) { // cycle through letters of word
					words[word_count][j] = '\0'; // nullify characters
					i=size; // end loop
				}
			}
		}
		if(strcmp(words[word_count], ".") == 0) break; // if user inputs a period end loop
		if(word_count == 19) { // if user inputs 20 words end loop
			word_count++; // update word count before exitting
			break;
		}
		word_count++; // update word count and continue
		if(subtract == true) { // if flag was set to true...
			word_count--; // subtract off word count
			subtract = false; // reset flag to false
		}
	}
	return word_count; // return total number of words
}

void to_caps(char words[][size], int word_count) { // FUNCTION TO CHANGE WORDS TO CAPS
	for(int w=0; w<word_count; w++) { // cycle thorugh all words
		for(int l=0; l<strlen(words[w]); l++) { // cycle through all letters
			words[w][l] = toupper(words[w][l]); // change each character to uppercase
		}
	}
}

void sort_words(char old_words[][size], char words[][size], int word_count) { // FUNCTION TO SORT WORDS IN ORDER OF LENGTH
	int count = 0;
	for(int i=size; i>1; i--) { // i = number of letters, start at maximmum possible
		for(int j=0; j<word_count; j++) { // cycle through words
			if(strlen(old_words[j]) == i) { // if word contains i amount of letters, add to new word array
				strcpy(words[count], old_words[j]);
				count++; // update counter
			}
		}
	}
}

void create_board(char board[][size]) { // FUNCTION TO FILL EMPTY BOARD
	for(int x=0; x<size; x++) { // cycle through all rows and cols of board array
		for(int y=0; y<size; y++) {
			board[y][x] = '.'; // set each space to period
		}
	}
}

void display_board(char board[][size]) { // FUNCTION TO DISPLAY GAME BOARD
	printf("Solution:\n");
	printf("-----------------\n"); // display top frame
	for(int x=0; x<size; x++) { // cycle through cols
		printf("|"); // display left frame
		for(int y=0; y<size; y++) { // cycle through rows
			printf("%c", board[y][x]); // display indexed characters
		}
		printf("|\n"); // display right frame
	}
	printf("-----------------\n"); // display bottom frame
	printf("Crossword puzzle:\n");
	printf("-----------------\n"); // display top frame
	for(int i=0; i<size; i++) {
		printf("|"); // display left frame
		for(int j=0; j<size; j++) {
			if(board[j][i] == '.') { // replace periods with number signs
			printf("#");
			}
			else { // replace characters with spaces
				printf(" ");
			}
		}
		printf("|\n"); // display right frame
	}
	printf("-----------------\n"); // display bottom frame
}

void display_board2(char board[][size], FILE *fp) { // FUNCTION TO DISPLAY GAME BOARD IN OUTPUT FILE
	fprintf(fp, "Solution:\n");
	fprintf(fp, "-----------------\n"); // display top frame
	for(int x=0; x<size; x++) { // cycle through cols
		fprintf(fp, "|"); // display left frame
		for(int y=0; y<size; y++) { // cycle through rows
			fprintf(fp, "%c", board[y][x]); // display indexed characters
		}
		fprintf(fp, "|\n"); // display right frame
	}
	fprintf(fp, "-----------------\n"); // display bottom frame
	fprintf(fp, "Crossword puzzle:\n");
	fprintf(fp, "-----------------\n"); // display top frame
	for(int i=0; i<size; i++) {
		fprintf(fp, "|"); // display left frame
		for(int j=0; j<size; j++) {
			if(board[j][i] == '.') { // replace periods with number signs
			fprintf(fp, "#");
			}
			else { // replace characters with spaces
				fprintf(fp, " ");
			}
		}
		fprintf(fp, "|\n"); // display right frame
	}
	fprintf(fp, "-----------------\n"); // display bottom frame
}

void word_to_board(char words[][size], char board[][size], int word_count, int index_direction[word_count][3]) { // FUNCTION TO PLACE 1ST  WORD
	int original_index = 7-strlen(words[0])/2; // find coloumn index for first word
	int index = original_index;
	index_direction[0][0] = index; // first index is for the word; second index 0=col, 1=row, 2=direction (direction 1 is across and 2 is down)
	index_direction[0][1] = 7;
	index_direction[0][2] = 1;
	for(int i=0; i<strlen(words[0]); i++) { // cycle through letters of word 1
		board[index][7] = words[0][i]; // place letters in board
		index++; // update counter
	}
}

void clues(char words[][size], int word_count, int index_direction[word_count][3]) { // FUNCTION TO DISPLAY CLUES
	printf("Clues:\n");
	for(int o=1; o<3; o++) { // cycle for num of directions
		for(int k=0; k<15; k++) { // cycle for x coordinate
			for(int d=0; d<15; d++) { // cycle for y coordinate
				for(int i=0; i<word_count; i++) { // cycle through all words
					if(index_direction[i][2] == 1  && index_direction[i][0] == k && index_direction[i][1] == d && o == 1) { // if word is across...
						printf("%2d, %2d  Across  %s\n", index_direction[i][0], index_direction[i][1], strfry(words[i])); // display clue
					}
					else if(index_direction[i][2] == 2 && index_direction[i][0] == k && index_direction[i][1] == d && o == 2) { // if word is down...
						printf("%2d, %2d  Down    %s\n", index_direction[i][0], index_direction[i][1], strfry(words[i])); // display clue
					}
				}
			}
		}
	}
	printf("\n");
}

void clues2(char words[][size], int word_count, int index_direction[word_count][3], FILE *fp) { // FUNCTION TO DISPLAY CLUES IN OUTPUT FILE
	fprintf(fp, "Clues:\n");
	for(int o=1; o<3; o++) { // cycle for num of directions
		for(int k=0; k<15; k++) { // cycle for x coordinate
			for(int d=0; d<15; d++) { // cycle for y coordinate
				for(int i=0; i<word_count; i++) { // cycle through all words
					if(index_direction[i][2] == 1  && index_direction[i][0] == k && index_direction[i][1] == d && o == 1) { // if word is across...
						fprintf(fp, "%2d, %2d  Across  %s\n", index_direction[i][0], index_direction[i][1], strfry(words[i])); // display clue
					}
					else if(index_direction[i][2] == 2 && index_direction[i][0] == k && index_direction[i][1] == d && o == 2) { // if word is down...
						fprintf(fp, "%2d, %2d  Down    %s\n", index_direction[i][0], index_direction[i][1], strfry(words[i])); // display clue
					}
				}
			}
		}
	}
	fprintf(fp, "\n");
}

int read_file(char *argv[], char words[][size]) { // FUNCTION TO READ FILE
	char file_name[20];
	int word_count = 0;
	strcpy(file_name, argv[1]); // gets file name
	FILE *fp = fopen(file_name, "r"); // reads file
	if(!fp) { // test file name
		printf("Error: Invalid File Name\n"); // display error for invalid file
	}
	printf("\nAnagram Crossword Puzzle Generator\n");
	printf("----------------------------------\n");
	char string[size];
	bool skip = false;
	while(1) { // loops indefinetly
		fgets(string, size, fp); // read string
		string[strlen(string)-1] = '\0'; // eliminate null character
		printf("%s\n", string); // display words
		for(int i=0; i<strlen(string); i++) {
			if(isalpha(string[i]) == 0 && i!=0) { // display error for non alphabetical characters
				printf("Error: Word Ignored (Contains non alphabetical character(s))\n");
				skip = true;
			}
		}
		if(string[0] == '.') break; // when a period is read, break loop

		if(strlen(string) < 2) { // display error for strings of 1 character
			printf("Error: Word Ignored (Contains less than 2 characters)\n");
		}
		else if(skip) skip = false; // to skip else statement
		else {
			strcpy(words[word_count], string); // copy string to word array
			word_count++; // increase word count
		}
	}
	return word_count;
}

void all_words(int word_count, char words[][size], int index_direction[word_count][3], char board[][size]) { // FUNCTION TO ADD THE REST OF THE WORDS
	bool done = false;
	bool bounds = false; // set to false when make function
	int row, col, direction, word_length;
	int zeros1 = 0;
	int zeros2 = 100;

	while(!done) { // keeps looping in order to come back to skipped words and fit maximum amount of words
		for(int w=0; w<word_count; w++) { // cycles through words
			if(index_direction[w][2] == 0) { // check for words not in board
				for(int l=0; l<strlen(words[w]); l++) { // cycles through letters of words not in board
					for(int y=0; y<word_count; y++) { // cycles through words
						if(index_direction[y][2] == 1 || index_direction[y][2] == 2) { // if word is in board
							for(int u=0; u<strlen(words[y]); u++) { // cycle through letters of word in board
								if(words[y][u] == words[w][l]) { // if letters match
									if(index_direction[y][2] == 1) { // ADDING VERTICAL WORD
										
										col = index_direction[y][0]+u; // col index for first letter of new word
										row = index_direction[y][1]-l; // row index for first letter of new word
										direction = 2; // direction of new word, vertical
										word_length = strlen(words[w]); // word length of new word

										bounds = check_bounds(col, row, board, direction, word_length, l); // check bounds for new word
									
										if(bounds) { // if the bounds are valid, set index and direction of new word
											index_direction[w][2] = 2;
											index_direction[w][0] = col;
											index_direction[w][1] = row;
											for(int s=0; s<strlen(words[w]); s++) { // to print word in board
												board[col][row] = words[w][s];
												row++;
											}
											w=100; y=100; u=100; l=100; // end all for loops
										}
									}
									else if(index_direction[y][2] == 2) { // ADDING HORIZONTAL WORD

										col = index_direction[y][0]-l; // col index for first letter of new word
										row = index_direction[y][1]+u; // row index for first letter of new word
										direction = 1; // direction of new word, horizontal
										word_length = strlen(words[w]); // word length of new word

										bounds = check_bounds(col, row, board, direction, word_length, l); // checl bounds for new word
		
										if(bounds) { // if the bounds are valid, set index and direction of new word
											index_direction[w][2] = 1;
											index_direction[w][0] = col;
											index_direction[w][1] = row;
											for(int t=0; t<strlen(words[w]); t++) { // to print word in board
												board[col][row] = words[w][t];
												col++;
											}
											w=100; y=100; u=100; l=100; // end all for loops
										}
									}
								}
							}
						}
					}
				}
			}
		}
		zeros1 = 0;
		for(int v=0; v<word_count; v++) { // to end while loop, if no new word was added to board end while loop
			if(index_direction[v][2] == 0) {
				zeros1++;
			}
		}
		if(zeros1 == zeros2) { // compare number of words not in board before and after every iteration of while loop
			done = true; // if no new words were added set to true and end loop
		}
		zeros2 = zeros1;
	}
	if(zeros2 != 0) { // display note if words dont fit in board
		printf("Note: %d word(s) do not fit in the board.\n", zeros1);
	}
}

bool check_bounds(int col, int row, char board[][size], int direction, int word_length, int in) { // FUNCTION TO CHECK FOR BOUNDS
	bool bounds = true;
	if(direction == 2) { // for new vertical words
		for(int i=0; i<word_length; i++) { // go through its letters
			if(board[col-1][row+i] != '.' && i!=in) { // check left column
				bounds = false;
			}
			if(board[col+1][row+i] != '.' && i!=in) { // check right column
				bounds = false;
			}
		}
		if(board[col][row-1] != '.') { // check for character above
			bounds = false;
		}
		if(board[col][row+word_length] != '.') { // check for character below
			bounds = false;
		}
		if(row<0 || row+word_length-1>14) { // check array walls
			bounds = false;
		}
	}
	if(direction == 1) { // for new horizontal words
		for(int j=0; j<word_length; j++) {
			if(board[col+j][row-1] != '.' && j!=in) { // check top row
				bounds = false;
			}
			if(board[col+j][row+1] != '.' && j!=in) { // check bottom row
				bounds = false;
			}
		}
		if(board[col-1][row] != '.') { // check for character to the left
			bounds = false;
		}
		if(board[col+word_length][row] != '.') { // check for character to the right
			bounds = false;
		}
		if(col<0 || col+word_length-1>14) { // check array walls
			bounds = false;
		}
	}
	return bounds;
}
```

{{< /tab >}}
{{< /tabgroup >}}