Jasjot Sumal, ID 993402197     Alfonso Jiron, ID 994866648
//Sorry about submitting all the files separately instead of in a zip file
//I tried to zip it and I kept getting errors.

ECS 40
Winter, 2011

Homework 2: Conversion of mscp.c to mscp.cpp

The following changes were made to the original mscp.c file to convert it to
a c++ file of equivalent function:

(NOTE: line citations refer to the line number of the original file)

-line 117:
  This union of 'bk' and 'tt' does not allow variables 'bk' or 'tt' to be visible
  within other functions in c++. This is causing errors in funtions 'int cmp_bk',
  line 1199, and 'int search(int, int, int)', line 1678.
  These errors occur where ever 'bk' or 'tt' is used in these functions, and
  read as:
  "invalid use of incomplete type 'struct tt' ",
  "forward declaration of 'struct tt'",
  "invalid use of incomplete type 'const struct bk' ", and
  "forward declaration of  'const struct bk' "

  To fix this, do the following:
  Move the declarations or 'struct tt' and 'struct bk' from with the union to
  occur just before the union is declared. Remove the 'tt[CORE]' and 'bk[CORE]'
  text that occurs after the declaration, as this still needs to occur within
  the union.
  Next, give the union a name so it is no longer anonymous and declare the union
  so it is NOT static (so remove the 'static' part of the declaration). Within
  the union declaration call 'struct tt tt[CORE]' and 'struct bk bk[CORE]' as
  the function was previously doing. Make sure the 'core' is still included in
  the union declaration.
  So the end result should look like this:

  #define CORE (2048)
  static long booksize;                   /* Number of opening book entries */

          struct tt {                     /* Transposition table entry */
                  unsigned short hash;    /* - Identifies position */
                  short move;             /* - Best recorded move */
                  short score;            /* - Score */
                  char flag;              /* - How to interpret score */
                  char depth;             /* - Remaining search depth */
          };
          struct bk {                     /* Opening book entry */
                  unsigned long hash;     /* - Identifies position */
                  short move;             /* - Move for this position */
                  unsigned short count;   /* - Frequency */
          };


  /* Transposition table and opening book share the same memory */
  union U {
          struct tt                      /* Transposition table entry */
           tt[CORE];
          struct bk                      /* Opening book entry */
  } core;

  #define TTABLE (core.tt)
  #define BOOK (core.bk)

-line 749:
  In funciton 'int cmp_move(const void *ap, const void *bp)' 'const void'
  addresses are assigned to 'const struct move' pointers. Compiling in c++ gives
  an invalid conversion error, so the 'const void' addresses must be casted as
  'const struct move' pointers.
  Result should be:
  
  static int cmp_move(const void *ap, const void *bp)
  {
          const struct move *a = (const struct move*) ap;
          const struct move *b = (const struct move*) bp;

          if (a->prescore < b->prescore) return -1;
          if (a->prescore > b->prescore) return 1;
          return a->move - b->move; /* this makes qsort deterministic */
  }

-line 1052:
  The errors here are similar to the errors above, and are fixed the same way.
  In function 'int parse_move(char *line, int *num)' single character pointers
  (char*) are changed using string formatting throughout the function.
  This gives the following warning on lines 1052 and 1058:
  "deprecated conversion from string constant to 'char*'"
  Also "invalid conversion from 'const char*' 'char*'" errors given throughout
  this function as well.

  To fix these errors, cast the entries to the pointers as char* on which ever
  lines the errors are given. The result should be:

  static int parse_move(char *line, int *num)
{
        int                     move, matches;
        int                     n = 0;
        struct move             *m;
        char                    *piece = NULL;
        char                    *fr_file = NULL;
        char			*fr_rank = NULL;
        char                    *to_file = NULL;
        char                    *to_rank = NULL;
        char                    *prom_piece = NULL;
        char                    *s;

        while (xisspace(line[n]))      /* skip white space */
                n++;

        if (!strncmp(line+n, "o-o-o", 5)
        ||  !strncmp(line+n, "O-O-O", 5)
        ||  !strncmp(line+n, "0-0-0", 5)) {
                piece = (char*)"K"; fr_file = (char*)"e"; to_file = (char*)"c";
                n+=5;
        }
        else if (!strncmp(line+n, "o-o", 3)
        ||  !strncmp(line+n, "O-O", 3)
        ||  !strncmp(line+n, "0-0", 3)) {
                piece = (char*)"K"; fr_file = (char*)"e"; to_file = (char*)"g";
                n+=3;
        }
        else {
                s = (char*)strchr("KQRBNP", line[n]);
                if (s && *s) {
                        piece = s;
                        n++;
                }

                /* first square */

                s = (char*)strchr("abcdefgh", line[n]);
                if (s && *s) {
                        to_file = s;
                        n++;
                }

                s = (char*)strchr("12345678", line[n]);
                if (s && *s) {
                        to_rank = s;
                        n++;
                }

                if (line[n] == '-' || line[n] == 'x') {
                        n++;
                }

                s = (char*)strchr("abcdefgh", line[n]);
                if (s && *s) {
                        fr_file = to_file;
                        fr_rank = to_rank;
                        to_file = s;
                        to_rank = NULL;
                        n++;
                }

                s = (char*)strchr("12345678", line[n]);
                if (s && *s) {
                        to_rank = s;
                        n++;
                }

                if (line[n] == '=') {
                        n++;
                }
                s = (char*)strchr("QRBNqrbn", line[n]);
                if (s && *s) {
                        prom_piece = s;
                        n++;
                }
        }

-line 1199:
  Same error as on line 749; function 'static int cmp_bk(const void *ap, const
  void *bp)' gives invalid conversion errors again. 'ap' and 'bp' must be cast
  as 'const struct bk*'.
  Result should be:

  static int cmp_bk(const void *ap, const void *bp)
  {
          const struct bk *a = (const struct bk*) ap;
          const struct bk *b = (const struct bk*) bp;

          if (a->hash < b->hash) return -1;
          if (a->hash > b->hash) return 1;
          return (int)a->move - (int)b->move;
  }

-line 1263:
  Deprecated conversion warning in function 'static int book_move(void)'.
  "book" must be cast as 'char*'.
  Result:

  static int book_move(void)
  {
          int move = 0, sum = 0;
          long x = 0, y, m;
          char *seperator = (char *)"book:";
          unsigned long hash;

-line 1877:
  Deprecated conversion warning in function 'void cmd_new(char*)'.
  String in setup_board must be cast as 'char*'.
  Result:

  static void cmd_new(char *dummy)
  {
          setup_board((char *)"rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq -");
          load_book((char *)"book.txt");
          computer[0] = 0;
          computer[1] = 1;
          rnd_seed = time(NULL);
  }

-line 1900:
  The error message received is:
  "At global scale:
  1900: error: storage size of 'mscp_commands' isn't known"
  This problem is being caused by the forward delaration of 'mscp_commands'.

  Fixing this is a matter of rearranging the order in which 'mscp_commands' is
  formally declared and called.
  First, delete the line with the forward declaration (line 1900). Next, place
  the function 'static void cmd_help(char *dummy)' after the formal declaration
  of 'mscp_commands'. Now the compiler can allocate space for 'mscp_commands' as
  it creates the actual array, and then use the array in 'cmd_help'.
  Finally, declare the function 'cmd_help' before the declaration for
  'mscp_commands'.

  This will fix the problem with the forward declaration of 'mscp_commands',
  however you will now receive several deprecated conversion warnings
  concrening the text being inserted into each column of the array. These can
  be fixed by casting each piece of text being inserted as 'char*'. Do not cast
  'NULL' as 'char*' as it is not inserted like text.

  The result should look like this:

  static void cmd_help(char *dummy);

  struct command mscp_commands[] = {
   { (char*)"help",      cmd_help,       (char*)"show this list of commands"            },
   { (char*)"bd",        cmd_bd,         (char*)"display board"                         },
   { (char*)"ls",        cmd_list_moves, (char*)"list moves"                            },
   { (char*)"new",       cmd_new,        (char*)"new game"                              },
   { (char*)"go",        cmd_go,         (char*)"computer starts playing"               },
   { (char*)"test",      cmd_test,       (char*)"search (depth)"                        },
   { (char*)"quit",      cmd_quit,       (char*)"leave chess program"                   },
   { (char*)"sd",        cmd_set_depth,  (char*)"set maximum search depth (plies)"      },
   { (char*)"both",      cmd_both,       (char*)"computer plays both sides"             },
   { (char*)"force",     cmd_force,      (char*)"computer plays neither side"           },
   { (char*)"white",     cmd_white,      (char*)"set computer to play white"            },
   { (char*)"black",     cmd_black,      (char*)"set computer to play black"            },
   { (char*)"book",      cmd_book,       (char*)"lookup current position in book"       },
   { (char*)"undo",      cmd_undo,       (char*)"undo move"                             },
   { (char*)"quit",      cmd_quit,       (char*)"leave chess program"                   },
   { (char*)"xboard",    cmd_xboard,     (char*)"switch to xboard mode"                 },
   { (char*)"fen",       cmd_fen,        (char*)"setup new position"                    },
   { NULL,        cmd_default,           (char*)"enter moves in algebraic notation"     },
  };


  static void cmd_help(char *dummy)
  {
          struct command *c;

          puts("commands are:");
          c = mscp_commands;
          do {
                  printf("%-8s - %s\n", c->name ? c->name : "", c->help);
          } while (c++->name != NULL);
  }

-line 1964:
	The warning was a bad comparison between signed and unsigned int:
 In function 'int main()':
mscp.cpp:1964: warning: comparison between signed and unsigned integer expressions

	Fix: added a cast to the unsigned int to make it signed. 

 for (i=0; i<(signed)sizeof(zobrist); i++) {
                ( (byte*)zobrist )[i] = rnd() & 0xff;
        }

