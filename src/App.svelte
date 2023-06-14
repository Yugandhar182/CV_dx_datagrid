<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let isCVUploadPopupVisible = false;
	let selectedRowData = null;
	
	 // Add a variable to store the selected CV identifier
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
        alert("file uploaded successfully");
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
  
	function handleClose() {
	  // Perform close logic
	  console.log("Close clicked");
  
	  // Close the CV upload popup or CV view popup
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
        width: 3000,
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
              console.log(cvData);
              const cvId = cvData.internal.cloudFile.id;
              console.log(cvData.internal.cloudFile);
              console.log(cvId);

              const downloadLink = `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${cvId}&apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`;
              window.open(downloadLink);
            } else {
              alert("Failed to fetch CV file.");
            }
          });
          container.appendChild(downloadButton);

          const viewButton = document.createElement("button");
          viewButton.classList.add("btn", "btn-success", "mr-2");
          viewButton.innerText = "View CV";
          viewButton.addEventListener("click", async () => {
            const cvResponse = await fetch(
              `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
            );
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
              alert("Failed to fetch CV.");
            }
          });
          container.appendChild(viewButton);
            

        	const cvUploadButton = document.createElement("button");
				cvUploadButton.innerText = "CV Upload";
				cvUploadButton.classList.add("btn", "btn-success", "mr-2");
				cvUploadButton.addEventListener("click", function () {
				  const rowData = options.data;
				  selectedRowData = rowData;
				  isCVUploadPopupVisible = true;
          
				});
        container.appendChild(cvUploadButton);
          
        },
       
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
            confirmDeleteMessage: "Are you sure you want to delete  record?",
          },
        },
        paging: {
          pageSize:20,
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

  .popup-overlay {
	  position: fixed;
	  top: 0;
	  left: 0;
	  right: 0;
	  bottom: 0;
	  background-color: rgba(0, 0, 0, 0.5);
	  display: flex;
	  justify-content: center;
	  align-items: center;
	}
  
	.popup-content {
	  background-color: white;
	  padding: 20px;
	  border-radius: 4px;
	}
</style>

<h1 style="color: blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>
{#if isCVUploadPopupVisible}
<div class="popup-overlay">
  <div class="popup-content">
  <h3>Upload CV</h3>
  <input
    type="file"
    on:change="{(event) => uploadCV(event.target.files[0])}"
    accept=".pdf,.doc,.docx"
  />
  <button on:click={handleSave} class="btn btn-primary">Save</button>
  <button on:click={handleClose} class="btn btn-secondary">Close</button>
  </div>
</div>
{/if}
