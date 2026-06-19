<!DOCTYPE html>
<html>
<head>
  <title>Reading Study</title>
  <script src="jatos.js"></script>

  <script src="https://cdn.jsdelivr.net/npm/jspsych@6.3.1/jspsych.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/jspsych@6.3.1/plugins/jspsych-html-keyboard-response.js"></script>
  <link href="https://cdn.jsdelivr.net/npm/jspsych@6.3.1/css/jspsych.css" rel="stylesheet" type="text/css">
  
  <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>

  <style>
    .experiment-container { text-align: left; max-width: 800px; margin: 0 auto; }
    .context-text { font-family: Arial, sans-serif; font-size: 24px; color: #333; margin-bottom: 40px; }
    .experiment-text { font-family: 'Courier New', Courier, monospace; font-size: 28px; }
    .upload-container { font-family: Arial, sans-serif; text-align: center; margin-top: 100px; padding: 30px; border: 3px dashed #bbb; border-radius: 10px; background-color: #f9f9f9; }
    .upload-btn { font-size: 18px; padding: 10px 20px; cursor: pointer; }
  </style>
</head>
<body>

  <div id="setup-screen" class="upload-container">
    <h2>Self-Paced Reading Setup</h2>
    <p>Select your <strong>stimuli.csv</strong> file from this folder to begin:</p>
    <input type="file" id="csv-selector" class="upload-btn" accept=".csv">
  </div>

<script>
  jatos.onLoad(function() {

    document.getElementById('csv-selector').addEventListener('change', function(e) {
      var file = e.target.files[0];
      if (!file) return;

      Papa.parse(file, {
        header: true,
        skipEmptyLines: true,
        dynamicTyping: true,
        complete: function(results) {
          if (results.data && results.data.length > 0) {
            document.getElementById('setup-screen').style.display = 'none';
            launchExperiment(results.data);
          } else {
            alert("Error: Your CSV file seems to be empty!");
          }
        }
      });
    });

    function launchExperiment(my_items) {
      var main_timeline = [];

      main_timeline.push({
        type: 'html-keyboard-response',
        stimulus: '<p style="font-family:Arial; font-size:20px;">Welcome! Press the <strong>SPACEBAR</strong> to advance through the experiment.</p>',
        choices: [' ']
      });

      function getCombinedScreenHTML(contextSentence, targetWords, currentWordIndex) {
        var display = [];
        for (var i = 0; i < targetWords.length; i++) {
          if (i === currentWordIndex) { display.push(targetWords[i]); } 
          else { display.push("-".repeat(targetWords[i].length)); }
        }
        var targetHTML = '<div class="experiment-text">' + display.join(" ") + '</div>';
        var contextHTML = '<div class="context-text">' + contextSentence + '</div>';
        return '<div class="experiment-container">' + contextHTML + targetHTML + '</div>';
      }

      var randomized_blocks = [];

      my_items.forEach(function(item) {
        var rawContext = "";
        var rawTarget = "";

        Object.keys(item).forEach(function(key) {
          var cleanKey = key.trim().toLowerCase();
          if (cleanKey === "context") { rawContext = item[key]; }
          if (cleanKey === "target") { rawTarget = item[key]; }
        });

        if (!rawTarget) return;

        var item_timeline = [];
        var words = String(rawTarget).split(" ");

        words.forEach(function(word, word_index) {
          item_timeline.push({
            type: 'html-keyboard-response',
            stimulus: function() {
              return getCombinedScreenHTML(rawContext, words, word_index);
            },
            choices: [' ']
          });
        });

        randomized_blocks.push({ timeline: item_timeline });
      });

      randomized_blocks.sort(function() { return Math.random() - 0.5; });
      randomized_blocks.forEach(function(block) { main_timeline.push(block); });

      main_timeline.push({
        type: 'html-keyboard-response',
        stimulus: '<div style="font-family:Arial; text-align:center;">' +
                  '<h2>Done! Thank you for participating.</h2>' +
                  '<p>Your data is being saved securely on our servers...</p>' +
                  '</div>',
        choices: jsPsych.NO_KEYS,
        on_start: function() {
          var finalResultCSV = jsPsych.data.get().csv();
          jatos.submitResultData(finalResultCSV, jatos.endStudy);
        }
      });

      jsPsych.init({
        timeline: main_timeline
      });
    }
  });
</script>
</body>
</html>
