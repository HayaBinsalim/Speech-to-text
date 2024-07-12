# Speech-to-text
# Description 
This project is a speech-to-text converter based on HTML, JavaScript, and PHP. 

## Front end
The HTML page is what the user will see. It includes a recording button and a textbox below it to show the converted text. The cammands below were used to build this page. 
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Speech to Text</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #b3cde0; /* colour of background */
            font-family: Arial, sans-serif;
        }

        .container {
            text-align: center;
            background: #fff;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        h1 {
            color: #333;
        }

        #record {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #fbb6c2; /* colour of button */
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s;
        }

        #record:hover {
            background-color: #ff99aa;
        }

        #output {
            margin-top: 20px;
            padding: 10px;
            background-color: #fff;
            border: 1px solid #ccc;
            width: 300px;
            text-align: center;
            border-radius: 5px;
            box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Speech to Text Converter</h1>
        <button id="record">Start Recording</button>
        <div id="output">Your speech will appear here</div>
    </div>
    <script src="app.js"></script>   
</body>
</html>
```

## Back end 
The language used for the back end here is javascript. It converts the speech recorded into text, display it on the textbox in the front end, and sends it to another location—PHP. The commands below are the ones used for this file. 
```
document.addEventListener('DOMContentLoaded', function() {
    const recordButton = document.getElementById('record');
    const outputDiv = document.getElementById('output');
    let recognition;

    if ('webkitSpeechRecognition' in window) {
        recognition = new webkitSpeechRecognition();
    } else if ('SpeechRecognition' in window) {
        recognition = new SpeechRecognition();
    } else {
        alert('Your browser does not support speech recognition. Please use Chrome or Firefox.');
    }

    if (recognition) {
        recognition.continuous = false;
        recognition.interimResults = false;

        recognition.onstart = function() {
            recordButton.textContent = 'Recording...';
        };

        recognition.onresult = function(event) {
            const transcript = event.results[0][0].transcript;
            outputDiv.textContent = transcript;
            saveTextToDatabase(transcript);
        };

        recognition.onerror = function(event) {
            console.error(event.error);
        };

        recognition.onend = function() {
            recordButton.textContent = 'Start Recording';
        };

        recordButton.addEventListener('click', function() {
            recognition.start();
        });
    }
});

function saveTextToDatabase(text) {
    fetch('save_text.php', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({ text: text })
    })
    .then(response => response.json())
    .then(data => {
        console.log('Success:', data);
    })
    .catch((error) => {
        console.error('Error:', error);
    });
}
```

## Storing in database 
We will be using PHP here. This part is where the text and the database meet. The text is sent by the back end to this file that will send it then to the databse and store it there. It will show “Logged successfully” for when the text is stored without errors. The commands below are the ines used for this file. 
```
<?php
header('Content-Type: application/json');

$servername = "localhost";
$username = "root";
$password = "";
$dbname = "speech_to_text";

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($conn->connect_error) {
    die(json_encode(['error' => 'Database connection failed: ' . $conn->connect_error]));
}

$data = json_decode(file_get_contents('php://input'), true);
$text = $data['text'];

if ($text) {
    $stmt = $conn->prepare("INSERT INTO texts (text, timestamp) VALUES (?, NOW())");
    $stmt->bind_param("s", $text);
    $stmt->execute();
    $stmt->close();
    echo json_encode(['message' => 'Your text is saved successfully!']);
} else {
    echo json_encode(['error' => 'You didn't provide any text!']);
}

$conn->close();
?>
```

## Results 
The figures below are screenshots of what the user sees. The page that asks them to record their speech and where it will appear as a text. I have a done a test for this by starting the project, saying the word “test”, and getting it written as a text in the texbox. 

 ![image](https://github.com/user-attachments/assets/983ff567-a1ba-480b-934e-3966735a98c1)

<br /> 

 ![image](https://github.com/user-attachments/assets/9c377d46-517a-40d0-b73d-a08509e9566d)


