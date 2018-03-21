# Coding Sample #
+ The following code sample is the implementation of the logic for an android app of the classic game Tic-Tac-Toe.
+ This app makes use of the minimax algorithm to implement a simple AI for the user to play against.
+ The explanation of the code is documented in the comments.

```

package com.kaleshsingh.tictactoe;

// Required imports for the app
import android.content.res.Resources;
import android.os.AsyncTask;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.GridLayout;
import android.widget.ImageView;
import android.widget.LinearLayout;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {
   // MainActivity is the MainThread and also the UI thread.

    static class Move {
    // A class to represeent a move.
        int row;
        int col;
    }

    private class BestMoveTask extends AsyncTask<char[][], Integer, Move> {

        /* Gets the best move for the computer using the minimax algorithm.
           This is done in a background thread since the minimax algorithm
           is recursive and therefore not "instantaneous". The new thread
           allows the main thread (UI thread) to continue running smoothly
           so as to ensure good user experience.

         */

        @Override
        protected Move doInBackground(char[][]... chars) {
            // Find the best move for the Computer
            return findBestMove(board);
        }

        @Override
        protected void onPostExecute(Move move) {
            super.onPostExecute(move);

            // Make the move
            int position = moveToPosition(move);
            String idString = "imageView" + position;
            Resources res = getResources();
            int id = res.getIdentifier(idString, "id", getApplicationContext()
                    .getPackageName());
            ImageView counter = findViewById(id);

            counter.setClickable(false);        // TODO: Test this

            // Move the counter off the screen
            counter.setTranslationY(-1000f);

            // Set the image source
            counter.setImageResource(R.drawable.o_small);

            // Make the best move for the computer
            board[move.row][move.col] = opponent;

            // End Player's turn
            whoseTurn = player;

            // Animate the counter onto the screen
            counter.animate().setStartDelay(1000).translationYBy(1000f).rotationBy(360)
                    .setDuration(1000).withEndAction(new Runnable() {
                @Override
                public void run() {
                    makeBoardClickable(board);

                    if (gameOver(board))
                        decideResult(board);

                }
            });
        }
    }

    private static char player = 'X';
    private static char opponent = 'O';
    private static char whoseTurn = player;
    private static char[][] board = new char[][]    {
                                                        {' ', ' ', ' '},
                                                        {' ', ' ', ' '},
                                                        {' ', ' ', ' '}
                                                    };

    private static int evaluate(char[][] board) {
        /* This function evaluates the game board
           If the maximizing player (X) has won the evaluated score is +10
           If the minimizing player (Y) has won the evaluated score is -10
           Otherwise the evaluated score is 0
         */
        // Check Rows for X or O victory
        for (int row = 0; row < 3; ++row) {
            if (board[row][0] == board[row][1] && board[row][1] == board[row][2]) {
                if (board[row][0] == player)
                    return +10;
                else if (board[row][0] == opponent)
                    return -10;
            }
        }

        // Check Columns for X or O victory
        for (int col = 0; col < 3; ++col) {
            if (board[0][col] == board[1][col] && board[1][col] == board[2][col]) {
                if (board[0][col] == player)
                    return +10;
                else if (board[0][col] == opponent)
                    return -10;
            }
        }

        // Check Diagonals for X or O victory
        if (board[0][0] == board[1][1] && board[1][1] == board[2][2]) {
            if (board[0][0] == player)
                return +10;
            else if (board[0][0] == opponent)
                return -10;
        }

        if (board[0][2] == board[1][1] && board[1][1] == board[2][0]) {
            if (board[0][2] == player)
                return +10;
            else if (board[0][2] == opponent)
                return -10;
        }

        // Else if none of them have won return 0
        return 0;
    }


    private static boolean isMovesLeft(char[][] board) {
        /* Checks if there are any moves left on the board */
        for (int i = 0; i < 3; ++i)
            for (int j = 0; j < 3; ++j)
                if (board[i][j] == ' ')
                    return true;
        return false;
    }

    private static int minimax(char[][] board, int depth, boolean isMax) {

        /* Recursively checks what the next best move depending on whose turn
           it is (i.e. maximizer's or minimizer's); assuming that both player's
           play optimally.

           It makes each of the possible next move's for the current player,
           then assuming each player plays's optimally from then on it return
           the best evaluated score of the final board states (which were
           determined recursively).
         */

        int score = evaluate(board);

        // If maximizer has won the game, return his/her evaluated score
        if (score == 10)
            return score - depth;			// minus depth to win fastest

        // If minimizer has won the game, return his//her evaluated score
        if (score == -10)
            return score + depth;			// add depth to prolong defeat

        // If there are no more moves and no winner then, it is a tie
        if (!isMovesLeft(board))
            return 0;

        // If it is mazimizer's move
        if (isMax) {
            int best = -1000;

            // Traverse all cells
            for (int i = 0; i < 3; ++i) {
                for (int j = 0; j < 3; ++j) {
                    // Check if cell is empty
                    if (board[i][j] == ' ') {
                        // Make the move
                        board[i][j] = player;

                        // Call minimax recursively and choose the maximum value
                        best = Math.max(best, minimax(board, depth + 1, !isMax));

                        // Undo the move
                        board[i][j] = ' ';
                    }
                }
            }
            return best;
        }

        // If it is minimizer's move
        else {
            int best = 1000;

            // Traverse all cells
            for (int i = 0; i < 3; ++i) {
                for (int j = 0; j < 3; ++j) {
                    // Check if cell is empty
                    if (board[i][j] == ' ') {
                        // Make the move
                        board[i][j] = opponent;

                        // Call minmax recursively and choose the minimum value
                        best = Math.min(best, minimax(board, depth + 1, !isMax));

                        // Undo the move
                        board[i][j] = ' ';
                    }
                }
            }
            return best;
        }
    }

    // Find the best possible move for the computer (Minimizer)
    private static Move findBestMove (char[][] board) {

        /* Returns the best move for the computer (minimizing player)
           using the minimax algorithm.
         */

        int bestVal = 1000;
        Move bestMove = new Move();
        bestMove.row = -1;
        bestMove.col = -1;

        // Traverse all cells, evaluate minimax function for all empty cells.
        // And return the cell with the optimal value.

        for (int i = 0; i < 3; ++i) {
            for (int j = 0; j < 3; ++j) {
                // Check if the cell is empty
                if (board[i][j] == ' ') {

                    // Make the move
                    board[i][j] = opponent;

                    // Compute the evaluation function for this move
                    int moveVal = minimax(board, 0, true);

                    // Undo the move
                    board[i][j] = ' ';

                    // If the value of the current move is less than the best value,
                    // then update bestVal and bestMove
                    if (moveVal < bestVal) {
                        bestMove.row = i;
                        bestMove.col = j;
                        bestVal = moveVal;
                    }
                }
            }
        }
        return bestMove;
    }

    private static void clearBoard(char[][] board) {
        /* Clears the game board */
        for (int i = 0; i < 3; ++i) {
            for (int j = 0; j < 3; ++j) {
                board[i][j] = ' ';
            }
        }
    }

    private static boolean gameOver(char[][] board) {
        /* Determines whether the game is over or not */
        int boardVal = evaluate(board);
        return boardVal == +10 || boardVal == -10 || !isMovesLeft(board);
    }

    private void decideResult(char[][] board) {
        /* Determines the result of the game i.e.
           whether the player won or the computer won
           or it was a draw.
         */

        // Make the board unclickable
        makeBoardUnClickable(board);

        TextView headingTextView = findViewById(R.id.headingTextView);
        TextView messageTextView = findViewById(R.id.messageTextView);

        int boardVal = evaluate(board);
        if (boardVal == +10) {
            System.out.println("Congratulation you won!!!");
            headingTextView.setText("Congratulations!");
            messageTextView.setText("\nYou won!\n");
        } else if (boardVal == -10) {
            System.out.println("Sorry you lost :(... Try again.");
            headingTextView.setText("Sorry :(");
            messageTextView.setText("\nYou lost.\n");
        } else {
            System.out.println("It's a draw!");
            headingTextView.setText("Draw");
            messageTextView.setText("\nIt's a draw...\n");
        }

        GridLayout gridLayout = findViewById(R.id.gridLayout);
        gridLayout.animate().setStartDelay(300).rotationBy(1080).setDuration(1500)
                .withEndAction(new Runnable() {
            @Override
            public void run() {
                LinearLayout linearLayout = findViewById(R.id.gameMessageLayout);
                linearLayout.setVisibility(View.VISIBLE);
            }
        });
    }

    private void makeBoardUnClickable(char[][] board) {
        /* Makes the game board unclickable */
        for (int i = 0; i < 3; ++i) {
            for (int j = 0; j < 3; ++j) {
                if (board[i][j] == ' ') {
                    Move move = new Move();
                    move.row = i;
                    move.col = j;
                    int position = moveToPosition(move);
                    String idString = "imageView" + position;
                    Resources res = getResources();
                    int id = res.getIdentifier(idString, "id", getApplicationContext()
                            .getPackageName());
                    ImageView counter = findViewById(id);
                    counter.setClickable(false);
                }
            }
        }
    }

    private void makeBoardClickable(char[][] board) {
        /* Makes the game board clickable */
        for (int i = 0; i < 3; ++i) {
            for (int j = 0; j < 3; ++j) {
                if (board[i][j] == ' ') {
                    Move move = new Move();
                    move.row = i;
                    move.col = j;
                    int position = moveToPosition(move);
                    String idString = "imageView" + position;
                    Resources res = getResources();
                    int id = res.getIdentifier(idString, "id", getApplicationContext()
                            .getPackageName());
                    ImageView counter = findViewById(id);
                    counter.setClickable(true);
                }
            }
        }
    }

    private static Move positionToMove(int position) {
        /* Converts a position (int) to a Move */
        int index = position - 1;
        Move move = new Move();
        move.row = index / 3;
        move.col = index % 3;
        return move;
    }



    private static int moveToPosition(Move move) {
        /* Converts a Move to a position (int) */
        return 3 * move.row + move.col + 1;
    }

    public void dropIn(View view) {
        /* The onClick handler the executes the player's move
           based on which valid position he clicked on the screen.
         */
        System.out.println("Clicked");

        makeBoardUnClickable(board);

        ImageView counter = (ImageView) view;

        // Move the counter off the screen
        counter.setTranslationY(-1000f);

        // Set the image source to X
        counter.setImageResource(R.drawable.x_small);

        counter.setClickable(false);        // TODO: Test this

        int position = Integer.parseInt(counter.getTag().toString());
        Move move = positionToMove(position);
        board[move.row][move.col] = player;

        // End Player's turn
        whoseTurn = opponent;

        // Animate the X counter onto the screen
        counter.animate().translationYBy(1000f).rotationBy(360).setDuration(1000)
                .withEndAction(new Runnable() {
            @Override
            public void run() {

                if (gameOver(board))
                    decideResult(board);

                if (isMovesLeft(board)) {
                    // If the game is not over and moves are left
                    // Determine and make the best move for the computer.
                    BestMoveTask task = new BestMoveTask();
                    task.execute(board);
                }
            }
        });
    }

    public void playGame(View view) {
        /*  Sets the game state and board to their starting states. */

        whoseTurn = player;     // Set the starting turn to player
        clearBoard(board);      // Clear the board

        LinearLayout linearLayout = findViewById(R.id.gameMessageLayout);
        linearLayout.setVisibility(View.INVISIBLE);

        Button startGameButton = (Button) view;
        startGameButton.setText("Play Again");

        // Remove all counters (X's or O's) from the screen
        GridLayout gridLayout = findViewById(R.id.gridLayout);
        for (int i = 0; i < gridLayout.getChildCount(); ++i) {
            ((ImageView) gridLayout.getChildAt(i)).setImageResource(0);
        }

        makeBoardClickable(board);
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //AppCompatDelegate.setCompatVectorFromResourcesEnabled(true);

        clearBoard(board);
        makeBoardUnClickable(board);
    }
}

```
