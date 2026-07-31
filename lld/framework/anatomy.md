Anatomy of an LLD Interview
TL;DR: Every LLD interview follows the same arc: scope the problem, discover entities, design classes top-down, implement core logic, then extend. Master the 5-step framework and you'll never freeze at the whiteboard again.

Anatomy of an LLD Interview
Anatomy of an LLD Interview

The 5-Step Framework
Most candidates dive straight into coding when given a design prompt. This is the fastest way to fail. LLD interviews reward structured thinking, and interviewers evaluate your process as much as your final design.

Here is the framework that works across every LLD problem, with realistic time allocation for a 40-45 minute round:

Step	Time	What You Do
1. Requirements	5 min	Ask clarifying questions. Write down exactly what the system must do. Define what's out of scope.
2. Entities	3 min	Extract nouns from requirements. Aggressively filter: most nouns are fields, not classes.
3. Class Design	10-15 min	Start from the orchestrator. Define class responsibilities, relationships, and public APIs.
4. Implementation	10 min	Write the core methods. Focus on the 2-3 most interesting classes, not all of them.
5. Extensibility	5 min	Discuss how the design handles new requirements. Show where it bends vs. where it breaks.
The remaining time is buffer for questions and discussion. You won't always hit these numbers exactly, but the ordering is non-negotiable. Skipping requirements to "save time" always costs more time later when you realize you built the wrong thing.

Interview tip: Announce each step as you enter it. "Let me start by clarifying the requirements" and "Now I'll identify the core entities" signals structure and gives the interviewer clear checkpoints.

What Interviewers Evaluate
Different companies weight these differently, but every LLD interview scores you on the same five dimensions:

Dimension	What They're Looking For	Red Flag
Problem Analysis	You ask smart questions, narrow scope, identify edge cases before designing	Diving into code without asking a single question
Class Design	Clean separation of concerns, clear ownership of state and behavior	God class that does everything, or 15 tiny classes that do nothing
Code Quality	Readable methods, meaningful names, proper encapsulation	Public fields everywhere, methods with 8 parameters, copy-pasted logic
Extensibility	Design accommodates new requirements without rewriting core logic	"We'd need to rewrite the whole thing" for a simple addition
Communication	You explain your thinking, discuss tradeoffs, respond to hints	Coding silently for 20 minutes, or ignoring interviewer suggestions
Depth over breadth. A strong deep-dive into 2-3 core classes beats a shallow overview of 10. If you have a ParkingLot orchestrator, a Ticket class, and a well-designed FeeCalculator, you've demonstrated more skill than someone who sketched out 8 classes with empty method stubs.

Regional Differences
LLD interviews are not the same everywhere. Knowing what to expect prevents wasted preparation.

US Big Tech (Google, Meta, Amazon):

Emphasis on reasoning and tradeoffs. They want to hear why you chose a HashMap over a List.
Often a discussion round: you design on a whiteboard or shared doc, write pseudocode or partial code.
Interviewers push back on your design to test flexibility. "What if we need to support 10,000 concurrent users?"
Pattern names (Strategy, Observer) are nice to know but never required. Correct application matters, not vocabulary.
India / Asia (service companies, product companies):

Patterns are often asked directly: "Implement the Observer pattern for a notification system."
Machine coding rounds are common: you write compilable code in an IDE within 60-90 minutes.
Code must run. Pseudocode is usually not accepted.
SOLID principles are frequently tested as standalone questions.
Startups (everywhere):

Practical focus. "How would you design the backend for our checkout flow?"
Less formal structure. The interviewer might give you a real problem from their codebase.
They care more about working code than perfect OOP.
Interview Variants
Variant	Format	What to Prioritize
Discussion round	Whiteboard or shared doc, pseudocode acceptable	Class relationships, API design, tradeoff discussion
Machine coding	IDE, compilable code, 60-90 min	Working implementation first, clean design second
Take-home	2-4 hours, submitted as a repo	Test coverage, documentation, production-quality code
Live coding	Screen share, real language, 45 min	Balance between clean design and working code
For discussion rounds, spend more time on Steps 1-3 (requirements, entities, class design) and less on Step 4 (implementation). For machine coding rounds, invert that: get something running first, then refactor toward clean design.

Skip Formal UML
Do not spend time drawing formal UML diagrams with strict notation, visibility symbols, and cardinality markers. UML was designed for a different era when inspecting or running code was expensive. That tradeoff no longer holds. Microsoft removed UML tooling from Visual Studio in 2016 because usage had effectively dropped to zero.

In an interview, simple boxes with arrows and labels communicate structure far better than formal UML. If an interviewer explicitly asks for UML, ask whether simplified class notation is acceptable — in most cases, it will be.

Interview tip: A simple list of classes with their state and methods communicates more clearly than a formal class diagram. Save your time for the design, not the notation.

Level Expectations
The same problem (say, "Design a Parking Lot") is evaluated differently depending on the role you're interviewing for:

Junior (0-2 years):

Produce a working solution that handles the core flow
Basic class separation (don't put everything in one class)
Handle the happy path correctly
Acceptable: some code duplication, simple data structures, not every edge case covered
Mid-level (2-5 years):

Clean separation of concerns across classes
Handle extensions without major rewrites (add a new vehicle type, new fee structure)
Proper encapsulation: private fields, meaningful public APIs
Expected: discuss at least one tradeoff, handle 2-3 edge cases
Senior (5+ years):

Proactively identify tradeoffs before the interviewer asks
Production thinking: thread safety, error handling, monitoring hooks
Design that anticipates changes the interviewer hasn't mentioned yet
Expected: lead the discussion, explain why alternatives were rejected, consider operational concerns
Interview tip: At the senior level, saying "I'm choosing a HashMap here because lookups are O(1) and we'll need frequent lookups by ticket ID, but if memory becomes a concern we could switch to a TreeMap and accept O(log n)" demonstrates more skill than silently picking the right data structure.

Minute-by-Minute: A Real Tic-Tac-Toe Interview
Here is exactly what a strong 45-minute LLD interview looks like. Every minute is accounted for. This is based on a real interview at a top-tier company.

Minute 0-1: Read the prompt, take a breath

The interviewer says: "Design a Tic-Tac-Toe game." You resist the urge to start drawing classes. Instead, you open a shared doc or pick up the marker and say:

"Before I start designing, let me make sure I understand the requirements."

Minutes 1-5: Scoping

Code
Copy
You:  "Is this 2 players on one machine, or networked?"
Them: "Same machine."

You:  "Standard 3x3, or should I think about NxN?"
Them: "Start with 3x3. We can discuss NxN at the end."

You:  "Win/draw only, or can a player resign?"
Them: "Win or draw."

You:  "Do I need a UI or just the game logic?"
Them: "Just the logic."

You:  "Should I handle replaying or just a single game?"
Them: "Single game is fine."
You write the requirements visibly:

Code
Copy
IN SCOPE: 2-player, 3x3, alternate turns, win/draw detection, 
          invalid move rejection
OUT OF SCOPE: UI, AI, networked play, undo, game history
What the interviewer writes on their rubric at this point:

Criteria	Score	Notes
Problem Analysis	4/5	Asked good narrowing questions. Identified scope boundaries.
Communication	4/5	Announced "let me start with requirements." Clear and structured.
Minutes 5-8: Entity Discovery

"The key entities I see are: a Game orchestrator, a Board that manages the grid, and Player. I'm going to demote Player to an enum -- all we need is X vs O. The board is a 3x3 grid of cell states."

You write:

Java
Copy
Classes: Game (orchestrator), Board
Enums:   PlayerMark (X, O), CellState (EMPTY, X, O), GameState (PLAYING, WIN, DRAW)
Minutes 8-22: Class Design + Implementation

You start with Game (the orchestrator):

Java
Copy
class Game {
    private Board board;
    private PlayerMark currentTurn;
    private GameState state;

    public Game() {
        this.board = new Board();
        this.currentTurn = PlayerMark.X;
        this.state = GameState.PLAYING;
    }

    public void makeMove(int row, int col) {
        if (state != GameState.PLAYING) {
            throw new IllegalStateException("Game is over");
        }
        board.place(row, col, currentTurn);

        if (board.checkWin(currentTurn)) {
            state = GameState.WIN;
        } else if (board.isFull()) {
            state = GameState.DRAW;
        } else {
            currentTurn = (currentTurn == PlayerMark.X) 
                ? PlayerMark.O : PlayerMark.X;
        }
    }

    public GameState getState() { return state; }
    public PlayerMark getCurrentTurn() { return currentTurn; }
}
Then Board:

Java
Copy
class Board {
    private final CellState[][] grid;
    private final int size;

    public Board() {
        this.size = 3;
        this.grid = new CellState[size][size];
        for (CellState[] row : grid) Arrays.fill(row, CellState.EMPTY);
    }

    public void place(int row, int col, PlayerMark mark) {
        if (row < 0 || row >= size || col < 0 || col >= size) {
            throw new IllegalArgumentException("Position out of bounds");
        }
        if (grid[row][col] != CellState.EMPTY) {
            throw new IllegalArgumentException("Cell already occupied");
        }
        grid[row][col] = CellState.valueOf(mark.name());
    }

    public boolean checkWin(PlayerMark mark) {
        CellState target = CellState.valueOf(mark.name());
        // Check rows, columns, diagonals
        for (int i = 0; i < size; i++) {
            if (allMatch(grid[i], target)) return true;
            if (columnMatches(i, target)) return true;
        }
        return diagonalMatches(target) || antiDiagonalMatches(target);
    }

    public boolean isFull() {
        for (CellState[] row : grid)
            for (CellState cell : row)
                if (cell == CellState.EMPTY) return false;
        return true;
    }
}
Minutes 22-28: Interviewer pushback

Interviewer: "What about NxN? How does your win detection change?"

"Currently checkWin iterates rows, columns, and diagonals in O(n^2). For NxN, I'd keep the same approach -- it's simple and correct. If we needed O(1) per-move checking, I'd maintain running counters for each row, column, and both diagonals. Each move increments or decrements the counter, and a win is when any counter hits n."

Java
Copy
// O(1) win detection for NxN
class Board {
    private final int[] rowCounts;     // +1 for X, -1 for O
    private final int[] colCounts;
    private int diagCount;
    private int antiDiagCount;

    public boolean placeAndCheckWin(int row, int col, PlayerMark mark) {
        int delta = (mark == PlayerMark.X) ? 1 : -1;
        rowCounts[row] += delta;
        colCounts[col] += delta;
        if (row == col) diagCount += delta;
        if (row + col == size - 1) antiDiagCount += delta;

        int target = size * delta;
        return rowCounts[row] == target || colCounts[col] == target
            || diagCount == target || antiDiagCount == target;
    }
}
Minutes 28-35: Edge cases and extensions

You proactively raise:

"What if someone calls makeMove after the game is over? I throw IllegalStateException."
"What if we wanted to add undo? I'd store a move history stack and pop the last move."
"For multiplayer networking, I'd extract a GameServer that holds the Game, receives moves via messages, and broadcasts board state."
Minutes 35-40: Discussion

The interviewer asks: "Why did you separate Board from Game? Why not put the grid inside Game?"

"Game manages the flow -- whose turn it is, whether the game is over. Board manages the grid -- placement rules, win detection. If I merge them, adding NxN support or a different board shape would mean modifying game flow code. Separation lets me swap in a HexBoard or a 3D Board without touching Game."

What the interviewer's final rubric looks like:

Criteria	Score	Notes
Problem Analysis	4/5	Good scoping. Identified boundaries before designing.
Class Design	5/5	Clean Game/Board separation. NxN extension was elegant.
Code Quality	4/5	Clear methods. Proper validation. Good naming.
Extensibility	5/5	Proactively discussed NxN optimization, undo, networking.
Communication	4/5	Narrated decisions throughout. Responded well to pushback.
Result: Strong Hire.

Common Mistakes at Each Phase
Phase	Mistake	What Happens	Recovery
Requirements	Not asking any questions	You build a networked multiplayer game when they wanted local 2-player	"Let me take a step back and clarify the requirements." It's never too late.
Entities	Making every noun a class (Player, Move, Cell, Row, Column, Diagonal...)	You spend 20 minutes wiring empty shells together	"Let me simplify. Most of these are fields, not classes." Delete aggressively.
Class Design	Starting with the smallest class instead of the orchestrator	Your classes don't fit together because you designed bottom-up	"Let me start from the top-level API and see what classes fall out."
Implementation	Trying to write every method perfectly	You run out of time with 2 classes done and 3 not started	"I'll write the skeleton here and implement the interesting methods."
Extensibility	Saying "I'd rewrite the whole thing"	Signals your design is rigid	Find the specific class or method that changes. Extension should be local.
How to Recover When You're Stuck
Every candidate gets stuck at some point. The difference between a hire and a reject is what you do next.

Stuck on class design:

"I'm not sure whether X should be its own class or a field on Y. Let me think about what state it manages... [pause]... Actually, it doesn't maintain any changing state, so I'll demote it to a field."

The interviewer sees you applying a reasoning framework, not just guessing.

Stuck on implementation:

"I know this method needs to check all rows, columns, and diagonals for a win. Let me write the row check first, and the column and diagonal checks follow the same pattern."

Break the problem down. Solve one piece, and the rest becomes clearer.

Stuck on an extension question:

"That's a good question. Let me trace through my current design and see where this new requirement would plug in... [traces]... It would go here, in this class, because that's where the relevant state lives."

The interviewer is testing whether your design is navigable, not whether you have a memorized answer.

The one thing you should never do:

Go silent for more than 30 seconds. If you're thinking, say what you're thinking. Interviewers cannot give you credit for thoughts they can't hear.

Junior vs Mid vs Senior: Same Problem, Different Performance
Here is how three different candidates handle the same Parking Lot prompt. The differences are instructive.

Junior (0-2 years): Passes but doesn't impress
Asks 1-2 questions, misses scope boundaries
Creates 6-7 classes, most with only getters
Gets the happy path working
When asked "what if we add motorcycles?" says "I'd add an if-else"
Code works but has some copy-paste patterns
Interviewer note: "Functional but mechanical. No tradeoff discussion."

Mid-level (2-5 years): Solid performance
Asks 4-5 targeted questions, writes requirements list
Creates 3-4 classes with clear responsibilities
Handles edge cases (lot full, vehicle already parked)
When asked "what if we add motorcycles?" points to VehicleType enum and says "add a new enum value and a spot size mapping"
Uses HashMap for O(1) lookups, explains why
Interviewer note: "Clean design, good communication. Handled extensions well."

Senior (5+ years): Drives the conversation
Asks scoping questions AND proactively calls out what they're excluding and why
Creates 2-3 core classes + identifies where Strategy pattern would go for fee calculation
Before the interviewer asks, mentions: "If we needed to support multiple fee structures, I'd inject a FeeStrategy here"
Discusses thread safety: "In a real system, enter() and exit() would need synchronization. I'll note that but not implement it to save time."
When pushed on a design choice, explains what alternatives were rejected and why
Interviewer note: "Led the discussion. Production mindset. Clear tradeoff analysis."

The key differences: juniors implement, mid-levels design, seniors anticipate. You can be at any level and pass -- the bar adjusts. But knowing what level the interviewer expects helps you calibrate.

Quick Recap
Concept	What it means	Why it matters
5-step framework	Requirements, Entities, Class Design, Implementation, Extensibility	Gives you a repeatable structure so you never freeze
Depth over breadth	2-3 well-designed classes beat 10 shallow ones	Interviewers evaluate quality of thinking, not quantity of classes
Know your variant	Discussion vs machine coding vs take-home	Adjusts where you spend your limited time
Level-appropriate	Junior = working, Mid = clean + extensible, Senior = proactive tradeoffs	Hit the bar for your level, don't under- or over-engineer
