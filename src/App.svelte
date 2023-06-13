<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];

  onMount(async () => {
    const response = await fetch(
      "https://api.recruitly.io/api/candidate?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA"
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
      // Add the file button column

      // Define other columns as needed
    ];

    const dataGrid = new DevExpress.ui.dxDataGrid(document.getElementById("dataGrid"), {
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
          confirmDeleteMessage: "Are you sure you want to delete this record?",
        },
      },
      paging: {
        pageSize: 10,
      },

      onInitialized: () => {
        
      },
    });

    // Function to handle file download
    async function downloadCV(cvid) {
  try {
    const response = await fetch(
      `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${cvid}&apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
    );

    if (response.ok) {
      // Extract the file name from the response headers
      const contentDisposition = response.headers.get("content-disposition");
      const fileName = contentDisposition
        ? contentDisposition.split("filename=")[1]
        : "CV_File";

      // Create a temporary download link and trigger the download
      const blob = await response.blob();
      const url = URL.createObjectURL(blob);
      const link = document.createElement("a");
      link.href = url;
      link.download = "filename";
      link.click();

      // Show success message
      alert("CV downloaded successfully!");
    } else {
      console.error("CV download failed.");
      // Handle the error accordingly
    }
  } catch (error) {
    console.error("CV download error:", error);
    // Handle the error accordingly
  }
}

    // Add a custom command column for the download button
    columns.push({
      caption: "Download",
      width: 100,
      cellTemplate: function (container, options) {
        const button = document.createElement("button");
        button.innerText = "Download";
        button.className = "btn btn-primary";
        button.addEventListener("click", () => {
          downloadCV(options.data.id); // Pass the appropriate ID for the download
        });
        container.appendChild(button);
      },
    });

    dataGrid.option("columns", columns);
  });
</script>

<style>
  #dataGrid {
    height: 400px;
  }
</style>

<h1 style="color:blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>
