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
          const downloadLink = document.createElement("a");
          downloadLink.href = `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`;
          downloadLink.target = "_blank";
          downloadLink.download = `CV_${cloudFileId.id}.txt`;
          downloadLink.innerText = "Download CV";
          downloadLink.addEventListener("click", async (event) => {
            event.preventDefault();
            const cvResponse = await fetch(downloadLink.href);
            if (cvResponse.ok) {
              const cvData = await cvResponse.json();
              const cloudFileId = cvData.cloudFileId;

              const fileDownloadUrl = `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${cloudFileId}&apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`;
              const fileResponse = await fetch(fileDownloadUrl);
              if (fileResponse.ok) {
                const fileBlob = await fileResponse.blob();
                const fileUrl = URL.createObjectURL(fileBlob);
                const fileLink = document.createElement("a");
                fileLink.href = fileUrl;
                fileLink.download = downloadLink.download;
                fileLink.click();
                URL.revokeObjectURL(fileUrl);
              } else {
                alert("Failed to fetch file.");
              }
            } else {
              alert("Failed to fetch CV file.");
            }
          });
          container.appendChild(downloadLink);

          const viewLink = document.createElement("a");
          viewLink.href = `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`;
          viewLink.target = "_blank";
          viewLink.innerText = "View CV";
          viewLink.addEventListener("click", async (event) => {
            event.preventDefault();
            const cvResponse = await fetch(viewLink.href);
            if (cvResponse.ok) {
              const cvData = await cvResponse.json();
              const cvHtml = cvData.html;
              if (cvHtml) {
                const cvWindow = window.open("", "_blank");
                cvWindow.document.write(cvHtml);
                cvWindow.document.close();
              } else {
                alert("CV file not found.");
              }
            } else {
              alert("Failed to fetch CV file.");
            }
          });
          container.appendChild(viewLink);
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
