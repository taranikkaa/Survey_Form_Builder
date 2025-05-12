# Survey Form Builder (ReactJS)
## Date:

## AIM
To create a Survey Form Builder using ReactJS..

## ALGORITHM
### STEP 1 Initialize States
mode ← 'build' (default mode)

questions ← [] (holds all survey questions)

currentQuestion ← { text: '', type: 'text', options: '' } (holds input while building)

editingIndex ← null (tracks which question is being edited)

responses ← {} (user responses to survey)

submitted ← false (indicates whether survey was submitted)

### STEP 2 Switch Modes
Toggle mode between 'build' and 'fill' when the toggle button is clicked.

Reset submitted to false when entering Fill Mode.

### STEP 3 Build Mode Logic
#### a. Add/Update Question
If currentQuestion.text is not empty:

If type is "radio" or "checkbox", split currentQuestion.options by commas → optionsArray.

Construct a questionObject:

If editingIndex is not null, replace question at that index.

Else, push questionObject into questions.

Reset currentQuestion and editingIndex.

#### b. Edit Question
Set currentQuestion to the selected question’s data.

Set editingIndex to the question’s index.

#### c. Delete Question
Remove the selected question from questions.

### STEP 4 Fill Mode Logic
#### a. Render Form
For each question in questions:

If type is "text", render a text input.

If type is "radio", render radio buttons for each option.

If type is "checkbox", render checkboxes.

#### b. Capture Responses
On input change:

For "text" and "radio", update responses[index] = value.

For "checkbox":

If option exists in responses[index], remove it.

Else, add it to the array.

### STEP 5 Submit Form
When user clicks "Submit":

Set submitted = true.

Display a summary using the questions and responses.

### STEP 6 Display Summary
Iterate over questions.

Display the response from responses[i] for each question:

If it's an array, join it with commas.

If empty, show "No response".


## PROGRAM
### Server builder.js:
```
import React, { useState } from "react";

function SurveyBuilder() {
  const [mode, setMode] = useState("build");
  const [questions, setQuestions] = useState([]);
  const [newQuestion, setNewQuestion] = useState({ text: "", type: "text", options: "" });
  const [responses, setResponses] = useState({});
  const [submitted, setSubmitted] = useState(false);

  const handleAddQuestion = () => {
    if (!newQuestion.text.trim()) return;

    const id = Date.now(); // unique ID for each question
    const formattedOptions = (newQuestion.type === "radio" || newQuestion.type === "checkbox")
      ? newQuestion.options.split(",").map(opt => opt.trim())
      : [];

    const question = {
      id,
      text: newQuestion.text,
      type: newQuestion.type,
      options: formattedOptions,
    };

    setQuestions([...questions, question]);
    setNewQuestion({ text: "", type: "text", options: "" });
  };

  const handleDelete = (id) => {
    setQuestions(questions.filter(q => q.id !== id));
  };

  const handleResponseChange = (id, value) => {
    setResponses({ ...responses, [id]: value });
  };

  const handleCheckboxToggle = (id, option) => {
    const prev = responses[id] || [];
    const updated = prev.includes(option)
      ? prev.filter(o => o !== option)
      : [...prev, option];
    setResponses({ ...responses, [id]: updated });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    setSubmitted(true);
  };

  return (
    <div style={{ maxWidth: "600px", margin: "0 auto", padding: "20px" }}>
      <button onClick={() => {
        setMode(mode === "build" ? "fill" : "build");
        setSubmitted(false);
        setResponses({});
      }} style={{ marginBottom: "20px", padding: "10px", backgroundColor: "#007bff", color: "#fff", border: "none" }}>
        Switch to {mode === "build" ? "Fill" : "Build"} Mode
      </button>

      {mode === "build" && (
        <div>
          <h2>Create Your Survey</h2>
          <div style={{ marginBottom: "10px" }}>
            <input
              type="text"
              placeholder="Question text"
              value={newQuestion.text}
              onChange={(e) => setNewQuestion({ ...newQuestion, text: e.target.value })}
              style={{ width: "100%", padding: "8px", marginBottom: "10px" }}
            />
            <select
              value={newQuestion.type}
              onChange={(e) => setNewQuestion({ ...newQuestion, type: e.target.value })}
              style={{ padding: "8px", marginRight: "10px" }}
            >
              <option value="text">Text</option>
              <option value="radio">Radio</option>
              <option value="checkbox">Checkbox</option>
            </select>
            {(newQuestion.type === "radio" || newQuestion.type === "checkbox") && (
              <input
                type="text"
                placeholder="Comma-separated options"
                value={newQuestion.options}
                onChange={(e) => setNewQuestion({ ...newQuestion, options: e.target.value })}
                style={{ width: "60%", padding: "8px", marginTop: "10px" }}
              />
            )}
          </div>
          <button onClick={handleAddQuestion} style={{ padding: "10px", backgroundColor: "green", color: "#fff" }}>
            Add Question
          </button>

          <ul style={{ marginTop: "20px" }}>
            {questions.map((q) => (
              <li key={q.id} style={{ marginBottom: "10px", padding: "10px", border: "1px solid #ccc" }}>
                <strong>{q.text}</strong> ({q.type})
                {q.options.length > 0 && (
                  <ul>
                    {q.options.map((opt, i) => <li key={i}>{opt}</li>)}
                  </ul>
                )}
                <button onClick={() => handleDelete(q.id)} style={{ marginTop: "5px", color: "red" }}>Delete</button>
              </li>
            ))}
          </ul>
        </div>
      )}

      {mode === "fill" && (
        <div>
          <h2>Fill the Survey</h2>
          {questions.length === 0 ? (
            <p>No questions created. Switch to build mode first.</p>
          ) : (
            <form onSubmit={handleSubmit}>
              {questions.map((q) => (
                <div key={q.id} style={{ marginBottom: "20px" }}>
                  <label><strong>{q.text}</strong></label><br />
                  {q.type === "text" && (
                    <input
                      type="text"
                      onChange={(e) => handleResponseChange(q.id, e.target.value)}
                      style={{ width: "100%", padding: "8px" }}
                    />
                  )}
                  {q.type === "radio" && q.options.map((opt, idx) => (
                    <div key={idx}>
                      <label>
                        <input
                          type="radio"
                          name={`question_${q.id}`}
                          value={opt}
                          onChange={() => handleResponseChange(q.id, opt)}
                        /> {opt}
                      </label>
                    </div>
                  ))}
                  {q.type === "checkbox" && q.options.map((opt, idx) => (
                    <div key={idx}>
                      <label>
                        <input
                          type="checkbox"
                          value={opt}
                          checked={responses[q.id]?.includes(opt) || false}
                          onChange={() => handleCheckboxToggle(q.id, opt)}
                        /> {opt}
                      </label>
                    </div>
                  ))}
                </div>
              ))}
              <button type="submit" style={{ padding: "10px", backgroundColor: "green", color: "#fff" }}>Submit</button>
            </form>
          )}
          {submitted && (
            <div style={{ marginTop: "30px" }}>
              <h3>Survey Results</h3>
              <ul>
                {questions.map((q) => (
                  <li key={q.id}>
                    <strong>{q.text}</strong>:{" "}
                    {Array.isArray(responses[q.id])
                      ? responses[q.id].join(", ")
                      : responses[q.id] || "No answer"}
                  </li>
                ))}
              </ul>
            </div>
          )}
        </div>
      )}
    </div>
  );
}

export default SurveyBuilder;

```
### App.js:
```
import React from "react";
import SurveyBuilder from "./SurveyBuilder";

function App() {
  return (
    <div>
      <h1 style={{ textAlign: "center", paddingTop: "20px" }}>Dynamic Survey Form Builder</h1>
      <SurveyBuilder />
    </div>
  );
}

export default App;
```


## OUTPUT

![image](https://github.com/user-attachments/assets/8d6c54a1-96f8-4471-ad61-8c4a96f6642d)




## RESULT
The program for creating Survey Form Builder using ReactJS is executed successfully.
