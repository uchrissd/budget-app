# Budget Application

**Description**

This budget app is designed to log deposits and expenses offline, regardless of whether the app is connected to wifi. When the user reconnects to the internet, the app will reflect the latest accounting once it updates the information in the database.

**User Story**

Given that I am someone who wants to track a budget, I would like the ability to use a web app that can store deposits and expenses both on and offline.

**Build status**

The build status is complete.

**Code style**

The budget app is written in Javascript and Node.js using an Express server, a Mongo database, and Mongoose to handle setting the database.

**Screenshots**

![Browser screenshot](app-screenshot.jpg)

**Code Example**

```javascript
//Index.js example
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

No installation necessary. Project is hosted here: https://ucsd-budget-app.herokuapp.com/
