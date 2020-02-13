# Budget Application

**Description**

Dungeon Bud is a web application that allows Dungeons & Dragons players to save characters and campaigns to a database. The goal of the application is to help players save their campaign and character information to a database. Users have unique profiles where they can add, edit or delete the campaigns and characters that they have added to the database. They can also add their characters to other player’s campaigns or add other player's characters to the campaigns they have created.

One thing our team noticed when developing this app is that there are no official, robust API’s for the Dungeons & Dragons game. If we were to scale this project up, taking on the task of building that API would be a goal of our team. It would allow for a more thorough campaign and character creation process.

**User Story**

Given that I am someone who plays Dungeons & Dragons regularily, I want to be able to store information about my characters and the campaigns I have created or am involved with in an online application.

**Build status**

The build status is complete.

**Code style**

Dungeon Bud is written in JavaScript. The back end of the application functions within the Node.js framework. We are using a MySql database with three tables: characters, campaigns and users. We used the Node.js Sequelize library to generate these tables.

Our application relies on the Node.js Express server library to handle our server and routing.

The front-end of the application uses jQuery for handling any dynamic HTML rendering via JavaScript as well as our API calls to the Dungeons & Dragons API for class and race information.

Our team used the Google Materialize library to design our grid system and some of the application’s styling, like buttons, drop-downs and modals. We also used customs CSS styling for other parts of the application, like fonts, logo and backgrounds.

**Screenshots**

![Browser screenshot](/Screenshots/app-screenshot.jpg)

**Code Example**

```javascript
//Create new character
let transactions = [];
let myChart;

fetch("/api/transaction")
  .then(response => response.json())
  .then(data => {
    // save db data on global variable
    transactions = data;
    populateTotal();
    populateTable();
    populateChart();
  });

function populateTotal() {
  // reduce transaction amounts to a single total value
  const total = transactions.reduce((total, t) => {
    return total + parseInt(t.value);
  }, 0);

  const totalEl = document.querySelector("#total");
  totalEl.textContent = total;
}

function populateTable() {
  const tbody = document.querySelector("#tbody");
  tbody.innerHTML = "";

  transactions.forEach(transaction => {
    // create and populate a table row
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td>${transaction.name}</td>
      <td>${transaction.value}</td>
    `;

    tbody.appendChild(tr);
  });
}

function populateChart() {
  // copy array and reverse it
  const reversed = transactions.slice().reverse();
  let sum = 0;

  // create date labels for chart
  const labels = reversed.map(t => {
    const date = new Date(t.date);
    return `${date.getMonth() + 1}/${date.getDate()}/${date.getFullYear()}`;
  });

  // create incremental values for chart
  const data = reversed.map(t => {
    sum += parseInt(t.value);
    return sum;
  });

  // remove old chart if it exists
  if (myChart) {
    myChart.destroy();
  }

  const ctx = document.getElementById("myChart").getContext("2d");

  myChart = new Chart(ctx, {
    type: "line",
    data: {
      labels,
      datasets: [
        {
          label: "Total Over Time",
          fill: true,
          backgroundColor: "#6666ff",
          data
        }
      ]
    }
  });
}

function sendTransaction(isAdding) {
  const nameEl = document.querySelector("#t-name");
  const amountEl = document.querySelector("#t-amount");
  const errorEl = document.querySelector("form .error");

  // validate form
  if (nameEl.value === "" || amountEl.value === "") {
    errorEl.textContent = "Forms must be completed";
    return;
  }

  // create record
  const transaction = {
    name: nameEl.value,
    value: amountEl.value,
    date: new Date().toISOString()
  };

  // if subtracting funds, convert amount to negative number
  if (!isAdding) {
    transaction.value *= -1;
  }

  // add to beginning of current array of data
  transactions.unshift(transaction);

  // re-run logic to populate ui with new record
  populateChart();
  populateTable();
  populateTotal();

  // also send to server
  fetch("/api/transaction", {
    method: "POST",
    body: JSON.stringify(transaction),
    headers: {
      Accept: "application/json, text/plain, */*",
      "Content-Type": "application/json"
    }
  })
    .then(response => response.json())
    .then(data => {
      if (data.errors) {
        errorEl.textContent = "Missing Information";
      } else {
        // clear form
        nameEl.value = "";
        amountEl.value = "";
      }
    })
    .catch(err => {
      // fetch failed, so save in indexed db
      saveRecord(transaction);

      // clear form
      nameEl.value = "";
      amountEl.value = "";
    });
}

document.querySelector("#add-btn").addEventListener("click", function(event) {
  event.preventDefault();
  sendTransaction(true);
});

document.querySelector("#sub-btn").addEventListener("click", function(event) {
  event.preventDefault();
  sendTransaction(false);
});
```

**Installation**

No installation necessary. Project is hosted here: https://dungeon-bud.herokuapp.com/

**Future development**

Our team identified several opportunities for further developing Dungeon Bud. These would include additional database tables to create more associations between users, developing our own Dungeons & Dragons API, a more robust character creation process, and the ability to build characters based on which version of the game you are using.

**Credits**

API used: https://www.dnd5eapi.co/ (for Dungeons & Dragons 5th Edition)
Libraries: Materialize CSS by Google
Developers: Courtney Tucker (Github: Ctucker9233), Brendan Berry (Github: berrybrendan), Christopher Underwood (Github: uchrissd), Blake Fogle (Github: foglebd)
