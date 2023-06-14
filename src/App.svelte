<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];

  onMount(async () => {
    const response = await fetch(
      "https://api.recruitly.io/api/candidate?apiKey=TEST9349C0221517DA4942E39B5DF18C68CDA154"
    );
    const responseData = await response.json();
    jsonData = responseData.data;

    gridData = jsonData.map((item) => ({
      id: item.id,
      firstName: item.firstName,
      surname: item.surname,
      email: item.email,
      mobile: item.mobile,
    }));

    const columns = [
      { dataField: "id", caption: "ID", width: 250 },
      { dataField: "firstName", caption: "Full Name", width: 200 },
      { dataField: "surname", caption: "Surname", width: 200 },
      { dataField: "email", caption: "Email", width: 200 },
      { dataField: "mobile", caption: "Mobile", width: 150 },
      {
        caption: "Actions",
        width: 250,
        cellTemplate: function (container, options) {
          const downloadButton = document.createElement("button");
          downloadButton.innerText = "Download CV";
          downloadButton.addEventListener("click", () => {
            const cvData = jsonData.find((item) => item.id === options.data.id);
            if (cvData && cvData.viewCV) {
              window.open(cvData.viewCV, "_blank");
            } else {
              alert("CV file not found.");
            }
          });
          container.appendChild(downloadButton);
        },
        width: 125,
      },
      {
        caption: "View CV",
        width: 125,
        cellTemplate: function (container, options) {
          const viewButton = document.createElement("button");
          viewButton.innerText = "View CV";
          viewButton.addEventListener("click", () => {
            const cvData = jsonData.find((item) => item.id === options.data.id);
            if (cvData && cvData.viewCV) {
              window.open(cvData.viewCV, "_blank");
            } else {
              alert("CV file not found.");
            }
          });
          container.appendChild(viewButton);
        },
        width: 125,
      },
    ];

    const dataGrid = new DevExpress.ui.dxDataGrid(
      document.getElementById("dataGrid"),
      {
        dataSource: gridData,
        columns: columns,
        showBorders: true,
        filterRow: {
          visible: true,
        },
        editing: {
          allowDeleting: true,
          allowAdding: true,
          allowUpdating: true,
          mode: "popup",
          form: {
            labelLocation: "top",
          },
          popup: {
            showTitle: true,
            title: "Row in the editing state",
          },
          texts: {
            saveRowChanges: "Save",
            cancelRowChanges: "Cancel",
            deleteRow: "Delete",
            confirmDeleteMessage:
              "Are you sure you want to delete this record?",
          },
        },
        paging: {
          pageSize: 10,
        },
      }
    );
  });
</script>

<style>
  #dataGrid {
    height: 400px;
  }
</style>

<h1 style="color: blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>
