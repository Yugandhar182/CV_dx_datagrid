<script>
  import { onMount, createEventDispatcher } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let isCVUploadPopupVisible = false;
  let selectedRowData = null;
  let isCVViewPopupVisible = false;
  let selectedCVId = null; // Add a variable to store the selected CV identifier

  async function uploadCV(file) {
    // Perform further actions with the uploaded file

    // Example: Update the backend API URL with the file upload
    const formData = new FormData();
    formData.append("file", file);

    if (selectedRowData) {
      const uploadCandidateId = selectedRowData.id; // Get the candidate ID from selectedRowData

      try {
        const response = await fetch(
          `https://api.recruitly.io/api/candidatecv/upload?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E&candidateId=${uploadCandidateId}`,
          {
            method: "POST",
            body: formData,
          }
        );

        if (response.ok) {
          console.log("CV uploaded successfully!");
          selectedCVId = await response.json(); // Store the uploaded CV identifier
          // Perform any additional actions upon successful upload
        } else {
          console.error("CV upload failed.");
          // Handle the error accordingly
        }
      } catch (error) {
        console.error("CV upload error:", error);
        // Handle the error accordingly
      }
    }

    // Close the CV upload popup
    isCVUploadPopupVisible = false;
  }

  async function downloadCV() {
    if (selectedCVId) {
      try {
        const response = await fetch(
          `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${selectedCVId}&apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`
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
          link.download = fileName;
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
  }

  async function viewCV() {
    if (selectedCVId) {
      try {
        const response = await fetch(
          `https://api.recruitly.io/api/candidatecv/${selectedCVId}?apiKey=TEST75806A94933FCBBD47039125432D3679DA21`
        );

        if (response.ok) {
          const contentType = response.headers.get("content-type");
          if (contentType && contentType.includes("application/json")) {
            const cvData = await response.json();
            console.log("CV data:", cvData);
            // Perform further actions with the CV data
          } else {
            console.error("CV view failed. Invalid response format.");
            // Handle the error accordingly
          }
        } else {
          console.error("CV view failed. Status:", response.status);
          // Handle the error accordingly
        }
      } catch (error) {
        console.error("CV view error:", error);
        // Handle the error accordingly
      }
    }
  }

  function openCVViewPopup() {
    // Perform any necessary actions before opening the popup

    // Set the visibility of the CV view popup to true
    isCVViewPopupVisible = true;

    // Fetch the CV data and perform further actions
    viewCV();
  }

  function handleSave() {
    // Perform save logic
    // In this case, we're updating the backend API URL in the handleSave function
    console.log("Save clicked");

    // Close the CV upload popup
    isCVUploadPopupVisible = false;
  }

  function handleClose() {
    // Perform close logic
    console.log("Close clicked");

    // Close the CV upload popup or CV view popup
    isCVUploadPopupVisible = false;
    isCVViewPopupVisible = false;
  }

  const dispatch = createEventDispatcher();

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
      cvUrl: item.cvUrl, // assuming cvUrl is the property containing the CV file URL
    }));

    const dataGrid = new DevExpress.ui.dxDataGrid(
      document.getElementById("dataGrid"),
      {
        dataSource: gridData,
        columns: [
          { dataField: "id", caption: "ID", width: 50 },
          { dataField: "firstName", caption: "First Name", width: 200 },
          { dataField: "surname", caption: "Surname", width: 200 },
          { dataField: "email", caption: "Email", width: 200 },
          { dataField: "mobile", caption: "Mobile", width: 150 },
          {
            caption: "Actions",
            width: 400,
            cellTemplate: function (container, options) {
              const cvUploadButton = document.createElement("button");
              cvUploadButton.innerText = "CV Upload";
              cvUploadButton.classList.add("btn", "btn-success", "mr-2");
              cvUploadButton.addEventListener("click", function () {
                const rowData = options.data;
                selectedRowData = rowData;
                isCVUploadPopupVisible = true;
              });

              const cvDownloadButton = document.createElement("button");
              cvDownloadButton.innerText = "CV Download";
              cvDownloadButton.classList.add("btn", "btn-info", "mr-2");
              cvDownloadButton.addEventListener("click", function () {
                const rowData = options.data;
                selectedRowData = rowData;
                selectedCVId = rowData.cvUrl; // Assuming cvUrl contains the CV identifier
                downloadCV();
              });

              const viewCVButton = document.createElement("button");
              viewCVButton.innerText = "View CV";
              viewCVButton.classList.add("btn", "btn-secondary");
              viewCVButton.addEventListener("click", function () {
                const rowData = options.data;
                selectedRowData = rowData;
                selectedCVId = rowData.cvUrl; // Assuming cvUrl contains the CV identifier
                openCVViewPopup();
                // Implement view CV logic here
                console.log("View CV clicked for row:", rowData);
              });

              container.appendChild(cvUploadButton);
              container.appendChild(cvDownloadButton);
              container.appendChild(viewCVButton);
            },
          },
        ],
      }
    );

    dataGrid.render();
  });
</script>

<style>
  #dataGrid {
    height: 400px;
  }
</style>

<h1>Recruitly Candidate CV Upload</h1>

<div id="dataGrid"></div>

{#if isCVUploadPopupVisible}
  <div class="modal" tabindex="-1" role="dialog" style="display: block;">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Upload CV</h5>
          <button type="button" class="close" on:click={handleClose}>
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          <input type="file" on:change={event => uploadCV(event.target.files[0])} accept=".pdf,.doc,.docx">
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-primary" on:click={handleSave}>Save</button>
          <button type="button" class="btn btn-secondary" on:click={handleClose}>Close</button>
        </div>
      </div>
    </div>
  </div>
{/if}

{#if isCVViewPopupVisible}
  <div class="modal" tabindex="-1" role="dialog" style="display: block;">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">View CV</h5>
          <button type="button" class="close" on:click={handleClose}>
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
        
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" on:click={handleClose}>Close</button>
        </div>
      </div>
    </div>
  </div>
{/if}
