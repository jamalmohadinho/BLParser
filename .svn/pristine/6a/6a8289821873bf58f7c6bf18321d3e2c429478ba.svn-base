import components.map.Map;
import components.program.Program;
import components.program.Program1;
import components.queue.Queue;
import components.simplereader.SimpleReader;
import components.simplereader.SimpleReader1L;
import components.simplewriter.SimpleWriter;
import components.simplewriter.SimpleWriter1L;
import components.statement.Statement;
import components.utilities.Reporter;
import components.utilities.Tokenizer;

/**
 * Layered implementation of secondary method {@code parse} for {@code Program}.
 *
 * @author Jamal Apicha and Aashi Sharma
 *
 *
 */
public final class Program1Parse1 extends Program1 {

    /*
     * Private members --------------------------------------------------------
     */

    /**
     * Checks whether a given identifier is a primitive instruction name.
     *
     * @param name
     *            the identifier to check
     * @return true if name is a primitive instruction and false otherwise
     */
    private static boolean isPrimitiveInstruction(String name) {
        return name.equals("move") || name.equals("turnleft") || name.equals("turnright")
                || name.equals("infect") || name.equals("skip");
    }

    /**
     * Parses a single BL instruction from {@code tokens} returning the
     * instruction name as the value of the function and the body of the
     * instruction in {@code body}.
     *
     * @param tokens
     *            the input tokens
     * @param body
     *            the instruction body
     * @return the instruction name
     * @replaces body
     * @updates tokens
     * @requires <pre>
     * [<"INSTRUCTION"> is a prefix of tokens]  and
     *  [<Tokenizer.END_OF_INPUT> is a suffix of tokens]
     * </pre>
     * @ensures <pre>
     * if [an instruction string is a proper prefix of #tokens]  and
     *    [the beginning name of this instruction equals its ending name]  and
     *    [the name of this instruction does not equal the name of a primitive
     *     instruction in the BL language] then
     *  parseInstruction = [name of instruction at start of #tokens]  and
     *  body = [Statement corresponding to the block string that is the body of
     *          the instruction string at start of #tokens]  and
     *  #tokens = [instruction string at start of #tokens] * tokens
     * else
     *  [report an appropriate error message to the console and terminate client]
     * </pre>
     */
    private static String parseInstruction(Queue<String> tokens, Statement body) {
        assert tokens != null : "Violation of: tokens is not null";
        assert body != null : "Violation of: body is not null";
        assert tokens.length() > 0 && tokens.front().equals("INSTRUCTION")
                : "" + "Violation of: <\"INSTRUCTION\"> is proper prefix of tokens";

        tokens.dequeue();

        Reporter.assertElseFatalError(tokens.length() > 0,
                "Expected identifier after INSTRUCTION");
        String instrName = tokens.dequeue();

        Reporter.assertElseFatalError(Tokenizer.isIdentifier(instrName),
                "Invalid instruction name: \"" + instrName + '"');
        Reporter.assertElseFatalError(!isPrimitiveInstruction(instrName),
                "Cannot redefine primitive instruction \"" + instrName + '"');

        Reporter.assertElseFatalError(
                tokens.length() > 0 && tokens.dequeue().equals("IS"),
                "Expected \"IS\" keyword");

        body.parseBlock(tokens);

        Reporter.assertElseFatalError(
                tokens.length() > 0 && tokens.dequeue().equals("END"),
                "Expected \"END\" keyword");

        Reporter.assertElseFatalError(tokens.length() > 0,
                "Expected identifier after END");
        if (tokens.front().equals("INSTRUCTION")) {
            tokens.dequeue();
            Reporter.assertElseFatalError(tokens.length() > 0,
                    "Expected identifier after END INSTRUCTION");
        }

        String closing = tokens.dequeue();
        Reporter.assertElseFatalError(closing.equals(instrName),
                "Instruction name at end must match start: expected \"" + instrName
                        + "\"");

        return instrName;
    }

    /*
     * Constructors -----------------------------------------------------------
     */

    /**
     * No-argument constructor.
     */
    public Program1Parse1() {
        super();
    }

    /*
     * Public methods ---------------------------------------------------------
     */

    @Override
    public void parse(SimpleReader in) {
        assert in != null : "Violation of: in is not null";
        assert in.isOpen() : "Violation of: in.is_open";
        Queue<String> tokens = Tokenizer.tokens(in);
        this.parse(tokens);
    }

    @Override
    public void parse(Queue<String> tokens) {
        assert tokens != null : "Violation of: tokens is not null";
        assert tokens.length() > 0
                : "" + "Violation of: Tokenizer.END_OF_INPUT is a suffix of tokens";

        Reporter.assertElseFatalError(tokens.dequeue().equals("PROGRAM"),
                "Expected keyword 'PROGRAM' at start of input");

        String progTitle = tokens.dequeue();
        Reporter.assertElseFatalError(Tokenizer.isIdentifier(progTitle),
                "Invalid program name, must be a valid identifier");

        Reporter.assertElseFatalError(tokens.dequeue().equals("IS"),
                "Expected keyword 'IS' after program name");

        Map<String, Statement> instructions = this.newContext();

        while (!tokens.front().equals("BEGIN")) {
            Statement instrBlock = this.newBody();
            String instrLabel = parseInstruction(tokens, instrBlock);

            // Check iff the instruction name is not a keyword or already used
            Reporter.assertElseFatalError(!Tokenizer.isKeyword(instrLabel),
                    "Instruction name cannot be a language keyword: " + instrLabel);
            Reporter.assertElseFatalError(!instructions.hasKey(instrLabel),
                    "Instruction name already used: " + instrLabel);

            instructions.add(instrLabel, instrBlock);
        }

        //Parsing the main program body
        Reporter.assertElseFatalError(tokens.dequeue().equals("BEGIN"),
                "Expected 'BEGIN' before main program body");

        Statement mainCode = this.newBody();
        mainCode.parseBlock(tokens);

        Reporter.assertElseFatalError(tokens.dequeue().equals("END"),
                "Expected 'END' to close the program");

        String closingName = tokens.dequeue();
        Reporter.assertElseFatalError(Tokenizer.isIdentifier(closingName),
                "Expected valid identifier after 'END'");
        Reporter.assertElseFatalError(progTitle.equals(closingName),
                "Program name at end does not match: expected '" + progTitle + "'");

        Reporter.assertElseFatalError(tokens.dequeue().equals(Tokenizer.END_OF_INPUT),
                "Unexpected content after program end");

        this.setName(progTitle);
        this.swapContext(instructions);
        this.swapBody(mainCode);
    }

    /*
     * Main test method -------------------------------------------------------
     */

    /**
     * Main method.
     *
     * @param args
     *            the command line arguments
     */
    public static void main(String[] args) {
        SimpleReader in = new SimpleReader1L();
        SimpleWriter out = new SimpleWriter1L();
        /*
         * Get input file name
         */
        out.print("Enter valid BL program file name: ");
        String fileName = in.nextLine();
        /*
         * Parse input file
         */
        out.println("*** Parsing input file ***");
        Program p = new Program1Parse1();
        SimpleReader file = new SimpleReader1L(fileName);
        Queue<String> tokens = Tokenizer.tokens(file);
        file.close();
        p.parse(tokens);
        /*
         * Pretty print the program
         */
        out.println("*** Pretty print of parsed program ***");
        p.prettyPrint(out);

        in.close();
        out.close();
    }

}
