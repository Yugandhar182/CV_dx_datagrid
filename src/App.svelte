<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  import CVView from "./CVView.svelte";

  let jsonData = [];
  let gridData = [];
  let isCVUploadPopupVisible = false;
  let isCVViewPopupVisible = false;
  let selectedRowData = null;
  let cvData = null;

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
          alert("File uploaded successfully");
          console.log("CV uploaded successfully!");
          console.log(uploadCandidateId);
          // Perform any additional actions upon successful upload
        } else {
          console.error("CV upload fail.");
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

  function handleSave() {
    // Perform save logic
    // In this case, we're updating the backend API URL in the handleSave function
    console.log("Save clicked");

    // Close the CV upload popup
    isCVUploadPopupVisible = false;
  }

  function handleViewCV() {
    isCVViewPopupVisible = true;
  }

  function handleClose() {
    isCVUploadPopupVisible = false;
    isCVViewPopupVisible = false;
  }

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
        width: 350,
        cellTemplate: function (container, options) {
          const downloadButton = document.createElement("button");
          downloadButton.classList.add("btn", "btn-success", "mr-2");
          downloadButton.innerText = "Download CV";
          downloadButton.addEventListener("click", async () => {
            const cvResponse = await fetch(
              `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
            );
            if (cvResponse.ok) {
              const cvData = await cvResponse.json();
              const cvFileUrl = cvData.file;
              window.open(cvFileUrl, "_blank");
            } else {
              alert("Failed to fetch CV.");
            }
          });
          container.appendChild(downloadButton);

          const viewButton = document.createElement("button");
          viewButton.classList.add("btn", "btn-primary", "mr-2");
          viewButton.innerText = "View CV";
          viewButton.addEventListener("click", async () => {
            const cvResponse = await fetch(
              `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
            );
            if (cvResponse.ok) {
              cvData = await cvResponse.json();
              handleViewCV();
            } else {
              alert("Failed to fetch CV.");
            }
          });
          container.appendChild(viewButton);

          const uploadButton = document.createElement("button");
          uploadButton.classList.add("btn", "btn-info", "mr-2");
          uploadButton.innerText = "Upload CV";
          uploadButton.addEventListener("click", () => {
            selectedRowData = options.data;
            isCVUploadPopupVisible = true;
          });
          container.appendChild(uploadButton);

          return container;
        },
      },
    ];

    new DevExpress.ui.dxDataGrid("#data-grid", {
      dataSource: gridData,
      columns: columns,
      columnAutoWidth: true,
    });
  });
</script>

<div class="container mt-4">
  <h2>Candidate List</h2>
  <div id="data-grid"></div>
</div>

<div
  class="popup-overlay"
  style="{isCVUploadPopupVisible || isCVViewPopupVisible ? 'display: flex' : 'display: none'}"
>
  {#if isCVUploadPopupVisible}
    <div class="popup-content">
      <h3>Upload CV</h3>
      <input type="file" on:change="{(e) => uploadCV(e.target.files[0])}" />
      <button on:click={handleSave} class="btn btn-primary">Save</button>
      <button on:click={handleClose} class="btn btn-secondary">Close</button>
    </div>
  {/if}

  {#if isCVViewPopupVisible}
    <CVView cvData={cvData} onClose={handleClose} />
  {/if}
</div>

<style>
  /* Add your custom styling for the popup overlay and content here */
  .popup-overlay {
    background-color: rgba(0, 0, 0, 0.5);
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 9999;
    align-items: center;
    justify-content: center;
  }

  .popup-content {
    background-color: white;
    padding: 20px;
    border-radius: 5px;
    max-width: 500px;
  }
</style>
