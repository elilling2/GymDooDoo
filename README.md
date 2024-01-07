<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GymDooDoo - Workout Timer</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 50px;
            position: relative;
        }

        h1 {
            font-size: 2em;
            margin-bottom: 20px;
        }

        #timer {
            font-size: 2em;
            margin-bottom: 20px;
        }

        #startBtn, #stopBtn, #resetBtn, #trackBtn {
            font-size: 1.5em;
            margin: 5px;
            padding: 10px 20px;
            cursor: pointer;
        }

        #workoutType, #weightInput {
            font-size: 1em;
            margin-top: 10px;
        }

        #otherInput {
            display: none;
            margin-top: 10px;
        }

        #history, #logSection, #weightSection {
            margin-top: 30px;
            text-align: left;
        }

        .workout-entry, .exercise-entry, .weight-entry {
            display: flex;
            justify-content: space-between;
            margin-bottom: 5px;
        }

        .delete-btn {
            background-color: red;
            color: white;
            border: none;
            padding: 5px 10px;
            cursor: pointer;
        }

        .log-date {
            font-size: 1.5em;
            font-weight: bold;
            margin-top: 20px;
        }

        /* Updated styles for the images */
        .gym-image {
            width: 150px; /* Adjust the width as needed */
            height: 150px; /* Set height equal to width for a square shape */
            border-radius: 10px; /* Optional: Add a border radius for a rounded corner */
            position: absolute;
        }

        #arnoldImage {
            top: 10px;
            right: 10px;
        }

        #physique1Image {
            top: 10px;
            right: 170px;
        }

        #physique2Image {
            top: 10px;
            left: 10px;
        }

        #physique3Image {
            top: 10px;
            left: 170px;
        }
    </style>
</head>
<body>
    <!-- Images added here -->
    <img class="gym-image" id="arnoldImage" src="images/Arnold image.jpg" alt="Arnold Schwarzenegger">
    <img class="gym-image" id="physique1Image" src="images/physique 1.jpg" alt="Physique 1">
    <img class="gym-image" id="physique2Image" src="images/physique 2.jpg" alt="Physique 2">
    <img class="gym-image" id="physique3Image" src="images/physique 3.jpg" alt="Physique 3">
    
 <h1>GymDooDoo by Easton Lilling</h1>

    <div id="timer">00:00:00</div>

    <label for="workoutType">Select workout type:</label>
    <select id="workoutType">
        <option value="push">Push Day</option>
        <option value="pull">Pull Day</option>
        <option value="legs">Legs</option>
        <option value="other">Other</option>
    </select>

    <div id="otherInput">
        <label for="other">Specify:</label>
        <input type="text" id="other" placeholder="Enter workout type">
    </div>

    <button id="startBtn" onclick="startTimer()">Start</button>
    <button id="stopBtn" onclick="stopTimer()">Stop</button>
    <button id="resetBtn" onclick="resetTimer()">Reset</button>
    <button id="trackBtn" onclick="trackWorkout()">Track</button>

    <div id="history">
        <h2>Workout History</h2>
        <ul id="historyList"></ul>
    </div>

    <div id="logSection">
        <h2>Exercise Log</h2>
        <label for="exerciseCategory">Select exercise category:</label>
        <select id="exerciseCategory" onchange="showExerciseOptions()">
            <option value="chest">Chest</option>
            <option value="triceps">Triceps</option>
            <option value="shoulders">Shoulders</option>
            <option value="back">Back</option>
            <option value="biceps">Biceps</option>
        </select>

        <div id="exerciseOptions" style="display: none;">
            <label for="specificExercise">Select specific exercise:</label>
            <select id="specificExercise"></select>

            <label for="reps">Reps:</label>
            <input type="number" id="reps" min="1" placeholder="Enter number of reps">

            <label for="weight">Weight (lbs):</label>
            <input type="number" id="weight" min="1" placeholder="Enter weight used">

            <button onclick="logExercise()">Log Exercise</button>
        </div>

        <div id="exerciseLog">
            <!-- Exercise log will be displayed here -->
        </div>
    </div>

    <div id="weightSection">
        <h2>Weight Log</h2>
        <label for="weightInput">Enter your weight (lbs):</label>
        <input type="number" id="weightInput" min="1" placeholder="Enter your weight">
        <button onclick="logWeight()">Log Weight</button>

        <div id="weightLog">
            <!-- Weight log will be displayed here -->
        </div>
    </div>

    <script>
        let timer;
        let startTime;
        let running = false;
        let workoutHistory = JSON.parse(localStorage.getItem('workoutHistory')) || [];
        let exerciseLog = JSON.parse(localStorage.getItem('exerciseLog')) || [];
        let weightLog = JSON.parse(localStorage.getItem('weightLog')) || [];

        window.onload = updateHistory;
        window.onload = updateExerciseLog;
        window.onload = updateWeightLog;

        function startTimer() {
            if (!running) {
                startTime = new Date().getTime();
                timer = setInterval(updateTimer, 1000);
                running = true;
            }
        }

        function stopTimer() {
            if (running) {
                clearInterval(timer);
                running = false;
            }
        }

        function resetTimer() {
            clearInterval(timer);
            running = false;
            document.getElementById('timer').innerHTML = '00:00:00';
        }

        function trackWorkout() {
            if (running) {
                stopTimer();
            }

            const workoutType = document.getElementById('workoutType').value;
            const workoutDate = new Date().toLocaleDateString();
            const workoutTime = calculateWorkoutTime();

            let workoutRecord;

            if (workoutType === 'other') {
                const otherInput = document.getElementById('other').value;
                workoutRecord = `${workoutDate} - ${workoutType} (${otherInput}) - ${workoutTime}`;
            } else {
                workoutRecord = `${workoutDate} - ${workoutType} - ${workoutTime}`;
            }

            workoutHistory.unshift(workoutRecord); // Add to the beginning of the array

            // Save the workout history to localStorage
            localStorage.setItem('workoutHistory', JSON.stringify(workoutHistory));

            // Display the workout history
            updateHistory();
        }

        function updateTimer() {
            const currentTime = new Date().getTime();
            const elapsedTime = currentTime - startTime;
            const hours = Math.floor(elapsedTime / 3600000);
            const minutes = Math.floor((elapsedTime % 3600000) / 60000);
            const seconds = Math.floor((elapsedTime % 60000) / 1000);

            document.getElementById('timer').innerHTML = `${formatTime(hours)}:${formatTime(minutes)}:${formatTime(seconds)}`;
        }

        function formatTime(time) {
            return time < 10 ? `0${time}` : time;
        }

        function updateHistory() {
            const historyList = document.getElementById('historyList');
            historyList.innerHTML = '';
            
            workoutHistory.forEach((workoutRecord, index) => {
                const listItem = document.createElement('li');
                listItem.classList.add('workout-entry');

                const deleteBtn = document.createElement('button');
                deleteBtn.classList.add('delete-btn');
                deleteBtn.textContent = 'Delete';
                deleteBtn.onclick = () => deleteWorkoutEntry(index);

                listItem.innerHTML = workoutRecord.replace(/(\b\w)/g, char => char.toUpperCase());
                listItem.appendChild(deleteBtn);

                historyList.appendChild(listItem);
            });
        }

        function deleteWorkoutEntry(index) {
            workoutHistory.splice(index, 1);

            // Save the updated workout history to localStorage
            localStorage.setItem('workoutHistory', JSON.stringify(workoutHistory));

            updateHistory();
        }

        function calculateWorkoutTime() {
            const currentTime = new Date();
            const elapsedTime = (currentTime - startTime) / 1000;
            const hours = Math.floor(elapsedTime / 3600);
            const minutes = Math.floor((elapsedTime % 3600) / 60);
            const seconds = Math.floor(elapsedTime % 60);

            return `${formatTime(hours)}:${formatTime(minutes)}:${formatTime(seconds)}`;
        }

        function showExerciseOptions() {
            const exerciseCategory = document.getElementById('exerciseCategory').value;
            const exerciseOptionsDiv = document.getElementById('exerciseOptions');
            const specificExerciseSelect = document.getElementById('specificExercise');
            const chestOptions = ['Bench Press', 'Incline Bench Press', 'Incline Dumbbell Bench Press', 'Dumbbell Bench Press', 'Chest Flies'];
            const tricepsOptions = ['Tricep Rope Pushdown', 'Dips', 'Close Grip Bench Press', 'Rope Extensions'];
            const shouldersOptions = ['Shoulder Press', 'Lateral Raises', 'Shoulder Shrugs', 'Dumbbell Front Lateral Raises'];
            const backOptions = ['Rows', 'Deadlift', 'Lat Pulldown', 'Cable Row', 'T-Bar Row'];
            const bicepsOptions = ['Hammer Curls', 'V-Bar Curls', 'Dumbbell Curls'];

            switch (exerciseCategory) {
                case 'chest':
                    fillOptions(specificExerciseSelect, chestOptions);
                    break;
                case 'triceps':
                    fillOptions(specificExerciseSelect, tricepsOptions);
                    break;
                case 'shoulders':
                    fillOptions(specificExerciseSelect, shouldersOptions);
                    break;
                case 'back':
                    fillOptions(specificExerciseSelect, backOptions);
                    break;
                case 'biceps':
                    fillOptions(specificExerciseSelect, bicepsOptions);
                    break;
                default:
                    break;
            }

            exerciseOptionsDiv.style.display = 'block';
        }

        function fillOptions(selectElement, options) {
            selectElement.innerHTML = '';
            options.forEach(option => {
                const formattedOption = option.replace(/([A-Z])/g, ' $1').trim();
                const optionElement = document.createElement('option');
                optionElement.value = option;
                optionElement.textContent = formattedOption;
                selectElement.appendChild(optionElement);
            });
        }

        function logExercise() {
            const exerciseCategory = document.getElementById('exerciseCategory').value;
            const specificExercise = document.getElementById('specificExercise').value;
            const reps = document.getElementById('reps').value;
            const weight = document.getElementById('weight').value;
            const logDate = new Date().toLocaleDateString();
            const logEntry = `${exerciseCategory} - ${specificExercise} - Reps: ${reps}, Weight: ${weight} lbs`;

            exerciseLog.unshift({ date: logDate, entry: logEntry }); // Add to the beginning of the array

            // Save the exercise log to localStorage
            localStorage.setItem('exerciseLog', JSON.stringify(exerciseLog));

            // Display the exercise log
            updateExerciseLog();
        }

        function deleteExerciseEntry(index) {
            exerciseLog.splice(index, 1);

            // Save the updated exercise log to localStorage
            localStorage.setItem('exerciseLog', JSON.stringify(exerciseLog));

            updateExerciseLog();
        }

        function updateExerciseLog() {
            const logSection = document.getElementById('exerciseLog');
            logSection.innerHTML = '';

            let currentDate = null;

            exerciseLog.forEach((log, index) => {
                if (log.date !== currentDate) {
                    const dateHeading = document.createElement('div');
                    dateHeading.classList.add('log-date');
                    dateHeading.textContent = log.date;
                    logSection.insertBefore(dateHeading, logSection.firstChild); // Insert at the beginning
                    currentDate = log.date;
                }

                const listItem = document.createElement('div');
                listItem.classList.add('log-entry');

                const deleteBtn = document.createElement('button');
                deleteBtn.classList.add('delete-btn');
                deleteBtn.textContent = 'Delete';
                deleteBtn.onclick = () => deleteExerciseEntry(index);

                listItem.innerHTML = log.entry.replace(/(\b\w)/g, char => char.toUpperCase());
                listItem.appendChild(deleteBtn);

                logSection.insertBefore(listItem, logSection.firstChild); // Insert at the beginning
            });
        }

        function logWeight() {
            const weightInput = document.getElementById('weightInput').value;
            const weightDate = new Date().toLocaleDateString();
            const weightEntry = `${weightDate} - Weight: ${weightInput} lbs`;

            weightLog.unshift(weightEntry); // Add to the beginning of the array

            // Save the weight log to localStorage
            localStorage.setItem('weightLog', JSON.stringify(weightLog));

            // Display the weight log
            updateWeightLog();
        }

        function updateWeightLog() {
            const weightLogSection = document.getElementById('weightLog');
            weightLogSection.innerHTML = '';

            weightLog.forEach(weightEntry => {
                const listItem = document.createElement('div');
                listItem.classList.add('weight-entry');

                const deleteBtn = document.createElement('button');
                deleteBtn.classList.add('delete-btn');
                deleteBtn.textContent = 'Delete';
                deleteBtn.onclick = () => deleteWeightEntry(weightLog.indexOf(weightEntry));

                listItem.innerHTML = weightEntry.replace(/(\b\w)/g, char => char.toUpperCase());
                listItem.appendChild(deleteBtn);

                weightLogSection.appendChild(listItem);
            });
        }

        function deleteWeightEntry(index) {
            weightLog.splice(index, 1);

            // Save the updated weight log to localStorage
            localStorage.setItem('weightLog', JSON.stringify(weightLog));

            updateWeightLog();
        }
    </script>
</body>
</html>
