
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Guess the Secret Sequence</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8; /* Light blue-gray background */
            display: flex;
            justify-content: center;
            align-items: flex-start; /* Align to top to allow scrolling if content is long */
            min-height: 100vh;
            padding: 20px;
            box-sizing: border-box;
        }
        .game-container {
            background-color: #ffffff;
            padding: 2.5rem; /* Increased padding */
            border-radius: 1.5rem; /* More rounded corners */
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1); /* Stronger shadow */
            max-width: 500px;
            width: 100%;
            text-align: center;
            border: 1px solid #e2e8f0; /* Light border */
        }
        input[type="number"] {
            width: 3.5rem; /* Fixed width for digits */
            height: 3.5rem; /* Fixed height for digits */
            text-align: center;
            font-size: 1.5rem; /* Larger font size */
            border: 2px solid #cbd5e1; /* Slightly darker border */
            border-radius: 0.75rem; /* Rounded input fields */
            margin: 0 0.3rem; /* Spacing between inputs */
            -moz-appearance: textfield; /* Remove number input arrows for Firefox */
        }
        input[type="number"]::-webkit-outer-spin-button,
        input[type="number"]::-webkit-inner-spin-button {
            -webkit-appearance: none; /* Remove number input arrows for Chrome/Safari */
            margin: 0;
        }
        button {
            padding: 0.8rem 1.8rem; /* Larger padding for buttons */
            font-size: 1.1rem; /* Larger font size for buttons */
            border-radius: 0.75rem;
            cursor: pointer;
            transition: background-color 0.3s ease, transform 0.1s ease;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        button:hover {
            transform: translateY(-2px);
        }
        button:active {
            transform: translateY(0);
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        }
        .btn-primary {
            background-color: #4f46e5; /* Indigo */
            color: white;
        }
        .btn-primary:hover {
            background-color: #4338ca; /* Darker indigo */
        }
        .btn-secondary {
            background-color: #6b7280; /* Gray */
            color: white;
        }
        .btn-secondary:hover {
            background-color: #4b5563; /* Darker gray */
        }
        .message-box {
            min-height: 2.5rem; /* Ensure consistent height */
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: 600;
            margin-bottom: 1.5rem;
            border-radius: 0.75rem;
            padding: 0.5rem;
        }
        .message-success {
            background-color: #d1fae5; /* Light green */
            color: #065f46; /* Dark green text */
        }
        .message-error {
            background-color: #fee2e2; /* Light red */
            color: #991b1b; /* Dark red text */
        }
        .guess-history-item {
            display: flex;
            justify-content.between;
            align-items: center;
            padding: 0.75rem 1rem;
            background-color: #f8fafc; /* Lighter background for history items */
            border-radius: 0.5rem;
            margin-bottom: 0.75rem;
            box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
        }
        .guess-digits {
            font-weight: 700;
            font-size: 1.1rem;
            color: #1a202c; /* Dark text */
            flex-grow: 1;
            text-align: left;
        }
        .guess-feedback {
            font-size: 0.95rem;
            color: #4a5568; /* Medium gray text */
            text-align: right;
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1 class="text-3xl font-extrabold text-gray-800 mb-6">Guess the Secret Sequence</h1>
        <p class="text-gray-600 mb-8">Try to guess the 4-digit number. Each digit is between 0 and 9.</p>

        <div id="messageBox" class="message-box text-sm mb-6">
            Enter your guess below!
        </div>

        <div class="flex justify-center mb-8">
            <input type="number" id="digit1" min="0" max="9" maxlength="1" oninput="this.value=this.value.slice(0,1)">
            <input type="number" id="digit2" min="0" max="9" maxlength="1" oninput="this.value=this.value.slice(0,1)">
            <input type="number" id="digit3" min="0" max="9" maxlength="1" oninput="this.value=this.value.slice(0,1)">
            <input type="number" id="digit4" min="0" max="9" maxlength="1" oninput="this.value=this.value.slice(0,1)">
        </div>

        <div class="flex justify-center space-x-4 mb-8">
            <button id="guessButton" class="btn-primary">Guess</button>
            <button id="newGameButton" class="btn-secondary">New Game</button>
        </div>

        <h2 class="text-xl font-bold text-gray-700 mb-4 text-left">Guess History</h2>
        <div id="guessHistory" class="bg-gray-50 p-4 rounded-lg border border-gray-200 min-h-[100px] max-h-[300px] overflow-y-auto">
            <!-- Guess history will be appended here -->
            <p class="text-gray-500 text-sm">No guesses yet.</p>
        </div>
    </div>

    <script>
        // Global variables for the game state
        let secretSequence = []; // Stores the 4-digit secret number
        let guessCount = 0;      // Tracks the number of guesses

        // Get references to DOM elements
        const digitInputs = [
            document.getElementById('digit1'),
            document.getElementById('digit2'),
            document.getElementById('digit3'),
            document.getElementById('digit4')
        ];
        const guessButton = document.getElementById('guessButton');
        const newGameButton = document.getElementById('newGameButton');
        const messageBox = document.getElementById('messageBox');
        const guessHistory = document.getElementById('guessHistory');

        /**
         * Generates a new 4-digit secret sequence.
         * Each digit is a random number between 0 and 9.
         */
        function generateSecretSequence() {
            secretSequence = []; // Clear previous secret
            for (let i = 0; i < 4; i++) {
                secretSequence.push(Math.floor(Math.random() * 10)); // Random digit from 0-9
            }
            console.log("Secret sequence (for debugging):", secretSequence.join('')); // Log for testing
        }

        /**
         * Displays a message in the message box.
         * @param {string} message - The message to display.
         * @param {string} type - 'success' or 'error' to apply specific styling.
         */
        function showMessage(message, type = '') {
            messageBox.textContent = message;
            messageBox.className = 'message-box text-sm mb-6'; // Reset classes
            if (type === 'success') {
                messageBox.classList.add('message-success');
            } else if (type === 'error') {
                messageBox.classList.add('message-error');
            }
        }

        /**
         * Handles the user's guess.
         */
        function handleGuess() {
            // Get the user's input digits
            const guess = digitInputs.map(input => parseInt(input.value));

            // Input validation: Check if all inputs are valid numbers
            if (guess.some(isNaN) || guess.length !== 4) {
                showMessage("Please enter 4 digits (0-9) for your guess.", 'error');
                return;
            }

            guessCount++; // Increment guess counter

            let bulls = 0;
            let cows = 0;

            // Create copies to avoid modifying original arrays during comparison
            const secretCopy = [...secretSequence];
            const guessCopy = [...guess];

            // 1. Calculate Bulls (correct digit in correct position)
            for (let i = 0; i < 4; i++) {
                if (guessCopy[i] === secretCopy[i]) {
                    bulls++;
                    // Mark as 'used' to prevent re-counting for cows
                    secretCopy[i] = -1; // Use a distinct value like -1 or null
                    guessCopy[i] = -2;
                }
            }

            // 2. Calculate Cows (correct digit in wrong position)
            for (let i = 0; i < 4; i++) {
                if (guessCopy[i] !== -2) { // If not already counted as a bull
                    const secretIndex = secretCopy.indexOf(guessCopy[i]);
                    if (secretIndex !== -1) {
                        cows++;
                        secretCopy[secretIndex] = -1; // Mark as 'used' in secret
                    }
                }
            }

            // Display feedback for the current guess
            const guessString = guess.join('');
            const feedbackMessage = `Bulls: ${bulls}, Cows: ${cows}`;
            addGuessToHistory(guessString, feedbackMessage);

            // Check if the guess is correct
            if (bulls === 4) {
                showMessage(`Congratulations! You guessed the sequence ${secretSequence.join('')} in ${guessCount} guesses!`, 'success');
                guessButton.disabled = true; // Disable guess button after winning
            } else {
                showMessage("Keep trying!", '');
            }

            // Clear input fields for the next guess
            digitInputs.forEach(input => input.value = '');
            digitInputs[0].focus(); // Focus on the first input for convenience
        }

        /**
         * Adds a guess and its feedback to the history display.
         * @param {string} guess - The user's guessed sequence.
         * @param {string} feedback - The Bulls and Cows feedback.
         */
        function addGuessToHistory(guess, feedback) {
            // Remove "No guesses yet." message if it exists
            if (guessHistory.querySelector('p.text-gray-500')) {
                guessHistory.innerHTML = '';
            }

            const historyItem = document.createElement('div');
            historyItem.classList.add('guess-history-item');
            historyItem.innerHTML = `
                <span class="guess-digits">${guess}</span>
                <span class="guess-feedback">${feedback}</span>
            `;
            guessHistory.prepend(historyItem); // Add to the top of the list
        }

        /**
         * Resets the game to a new state.
         */
        function startNewGame() {
            generateSecretSequence(); // Generate a new secret
            guessCount = 0;          // Reset guess counter
            guessHistory.innerHTML = '<p class="text-gray-500 text-sm">No guesses yet.</p>'; // Clear history
            showMessage("New game started! Guess the 4-digit sequence.", ''); // Reset message
            guessButton.disabled = false; // Enable guess button
            digitInputs.forEach(input => input.value = ''); // Clear input fields
            digitInputs[0].focus(); // Focus on the first input
        }

        // Event Listeners
        guessButton.addEventListener('click', handleGuess);
        newGameButton.addEventListener('click', startNewGame);

        // Allow pressing Enter key to guess when any input field is focused
        digitInputs.forEach(input => {
            input.addEventListener('keydown', (event) => {
                if (event.key === 'Enter') {
                    handleGuess();
                }
                // Automatically move focus to the next input field
                if (input.value.length === 1 && event.key !== 'Backspace' && event.key !== 'Delete' && event.key !== 'ArrowLeft' && event.key !== 'ArrowRight') {
                    const currentIndex = digitInputs.indexOf(input);
                    if (currentIndex < digitInputs.length - 1) {
                        digitInputs[currentIndex + 1].focus();
                    } else {
                        guessButton.focus(); // Focus on guess button after last digit
                    }
                }
            });
        });

        // Initialize the game when the page loads
        document.addEventListener('DOMContentLoaded', startNewGame);
    </script>
</body>
</html>
