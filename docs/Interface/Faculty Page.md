  

# Faculty Page Documentation: Course Slot and Clash Preferences

  


  

## 🧾 Overview

  

The page allows one to :

  

- Select a course from a list of existing courses (loaded from `config.json`)

- Choose preferred time slots for the selected course

- Mark other courses that should **not clash** with the selected course

- Submit preferences, which update the `config.json` file

  

---

  

## Setup : 

  

### 1. Environment Setup

  

Requisite dependencies :

  

```bash

pip  install  streamlit

```

  

### 2. Directory Structure

  

Project Folder structure is :

  

```

project/

│

├── app.py # Your main Streamlit script

├── footer.py # (Optional) External footer module (not currently used)

└── data/

└── config.json # Input/output config file

```

  

> Note: `footer.py` is imported but **not used in the code**. You can safely remove the import or define the module separately for reusable footer generation.

  

---

  


  

---

  

## File Dependencies

  

### `data/config.json`

  

The app depends on a configuration JSON file structured like this:

  

```json

{

"courses": [

{

"name": "CS101",

"full_name": "Intro to Programming",

"slots": [],

"clashes": []

}

],

"slots": [

{

"name": "Slot A",

"time": [

["09:00", "10:00"], // Monday

[], [], [], [], [], [] // Tuesday–Sunday

]

}

]

}

```

  

-  `courses[i]["slots"]` and `courses[i]["clashes"]` will be populated via the interaction.

-  `slots[i]["time"]` is a list of 7 elements (Monday–Sunday). Each element is a 2-item list: `[start_time, end_time]`, or `[]` if no slot on that day.

  

---

  

## Functional Flow

  

1.  **Course Selection:**

- User selects a course from a dropdown populated from `config.json`.

  

2.  **Slot Selection:**

- User selects time slots applicable for the course.

- Each selected slot shows the day-wise timing.

  

3.  **Clash Preferences:**

- User can select other courses that the selected course should not clash with.

  

4.  **Submit:**

- Updates the selected course’s `slots` and `clashes` in `config.json`.

- Shows a success message after submission.

  

5.  **Feedback:**

- Displays a footer with credits.

- If no course is selected, a warning is shown.

  

---

  
