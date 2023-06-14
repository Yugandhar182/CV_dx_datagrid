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
          const uploadButton = document.createElement("button");
          uploadButton.innerText = "Upload CV";
          uploadButton.addEventListener("click", async () => {
            const fileInput = document.createElement("input");
            fileInput.type = "file";
            fileInput.accept = ".pdf,.doc,.docx";
            fileInput.addEventListener("change", async (event) => {
              const file = event.target.files[0];
              const formData = new FormData();
              formData.append("file", file);

              const uploadResponse = await fetch(
                `https://api.recruitly.io/api/candidatecv/upload?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E&candidateId=${options.data.id}`,
                {
                  method: "POST",
                  body: formData,
                }
              );

              if (uploadResponse.ok) {
                alert("CV upload successfully.");
              } else {
                alert("Failed to upload CV.");
              }
            });

            fileInput.click();
          });
          container.appendChild(uploadButton);
        },
        width: 250,
      },
      // Add other columns as needed
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
        onInitialized: () => {},
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
