<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let cvDataMap = new Map();

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
          downloadButton.addEventListener("click", async () => {
            const cvData = cvDataMap.get(options.data.id);
            if (cvData) {
              const cvUrl = cvData.url;
              if (cvUrl) {
                const downloadLink = document.createElement("a");
                downloadLink.href = cvUrl;
                downloadLink.target = "_blank";
                downloadLink.download = `CV_${options.data.id}.pdf`;
                downloadLink.click();
              } else {
                alert("CV file not found.");
              }
            } else {
              alert("CV data not available.");
            }
          });
          container.appendChild(downloadButton);

          const viewButton = document.createElement("button");
          viewButton.innerText = "View CV";
          viewButton.addEventListener("click", async () => {
            const cvData = cvDataMap.get(options.data.id);
            if (cvData) {
              const cvHtml = cvData.html;
              if (cvHtml) {
                const cvWindow = window.open("", "_blank");
                cvWindow.document.write(cvHtml);
                cvWindow.document.close();
              } else {
                alert("CV file not found.");
              }
            } else {
              alert("CV data not available.");
            }
          });
          container.appendChild(viewButton);
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
      }
    );

    async function fetchCvData(candidateId) {
      const cvResponse = await fetch(
        `https://api.recruitly.io/api/candidatecv/${candidateId}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
      );
      if (cvResponse.ok) {
        const cvData = await cvResponse.json();
        cvDataMap.set(candidateId, cvData);
      } else {
        alert("Failed to fetch CV data.");
      }
    }

    for (const candidate of jsonData) {
      await fetchCvData(candidate.id);
    }
  });
</script>

<style>
  #dataGrid {
    height: 400px;
  }
</style>

<h1 style="color: blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>
