<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let isFileSelectionPopupOpen = false;
  let selectedFile = null;

  const handleFileSelection = (event) => {
    selectedFile = event.target.files[0];
  };

  const handleUploadCV = async (candidateId) => {
    isFileSelectionPopupOpen = true;
  };

  const handleFileUpload = async (options) => {
  if (options && options.data && options.data.id) {
    if (selectedFile) {
      const formData = new FormData();
      formData.append("file", selectedFile);

      try {
        const response = await fetch(
          `https://api.recruitly.io/api/candidatecv/upload?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E&candidateId=${options.data.id}`,
          {
            method: "POST",
            body: formData,
          }
        );

        if (response.ok) {
          alert("CV uploaded successfully.");
          // Perform any additional actions after successful upload
        } else {
          alert("Failed to upload CV.");
        }
      } catch (error) {
        console.error("Failed to upload CV:", error);
      }
    } else {
      alert("Please select a file to upload.");
    }

    // Reset file selection and close the popup
    selectedFile = null;
    isFileSelectionPopupOpen = false;
  } else {
    console.error("Invalid options or missing data.id property");
  }
};


  


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
        width: 400,
        cellTemplate: function (container, options) {
          const downloadButton = document.createElement("button");
          downloadButton.innerText = "Download CV";
          downloadButton.addEventListener("click", async () => {
            const cvResponse = await fetch(
              `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
            );
            if (cvResponse.ok) {
              const cvData = await cvResponse.json();
              const cvId = cvData.internal.cloudFile.id;
              const downloadLink = `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${cvId}&apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`;
              window.open(downloadLink);
              alert("Downloadded Successfully.");
            } else {
              alert("Failed to fetch CV file.");
            }
          });
          container.appendChild(downloadButton);

          const viewButton = document.createElement("button");
viewButton.innerText = "View CV";
viewButton.addEventListener("click", async () => {
  const cvResponse = await fetch(
    `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
  );
  if (cvResponse.ok) {
    const cvData = await cvResponse.json();
    const cvHtml = cvData.html;
    if (cvHtml) {
      const popupContainer = document.createElement("div");
      popupContainer.classList.add("popup-container");

      const popupContent = document.createElement("div");
      popupContent.classList.add("popup-content");

      const closeButton = document.createElement("button");
      closeButton.innerText = "Close";
      closeButton.addEventListener("click", () => {
        document.body.removeChild(popupContainer);
      });

      const cvContent = document.createElement("div");
      cvContent.innerHTML = cvHtml;

      popupContent.appendChild(closeButton);
      popupContent.appendChild(cvContent);
      popupContainer.appendChild(popupContent);

      document.body.appendChild(popupContainer);
    } else {
      alert("CV file not found.");
    }
  } else {
    alert("Failed to fetch CV file.");
  }
  
    });
          container.appendChild(viewButton);

          const uploadCVButton = document.createElement("button");
  uploadCVButton.innerText = "Upload CV";
  uploadCVButton.addEventListener("click", () => {
    handleUploadCV(options.data.id);
  });
  container.appendChild(uploadCVButton);

       

        
         
     
        
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
            confirmDeleteMessage:
              "Are you sure you want to delete this record?",
          },
        },
        paging: {
          pageSize: 10,
        },
        onRowInserting: async (e) => {
			console.log("Data being sent to API:", e.data);
			try {
			  const response = await fetch(
				"https://api.recruitly.io/api/candidate?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA",
				{
				  method: "POST",
				  headers: {
					"Content-Type": "application/json",
				  },
				  body: JSON.stringify(e.data),
				}
			  );
  
			  const responseData = await response.json();
			  if (response.ok) {
				
				e.data.firstName=responseData.fistName;
				gridData.push(e.data);
				dataGrid.refresh();
        alert("New data added successfully.");
			  } else {
				console.error("Failed to add record:", responseData.error);
			  }
			} catch (error) {
			  console.error("Failed to add record:", error);
			}
		  },
      onRowUpdating: async (e) => {
        try {
          console.log(e);
          var newData = {
            id: e.key.id,
            firstName: e.newData.firstName === undefined ? e.oldData.firstName : e.newData.firstName,
            surname: e.newData.surname === undefined ? e.oldData.surname : e.newData.surname,
            email: e.newData.email === undefined ? e.oldData.email : e.newData.email,
            mobile: e.newData.mobile === undefined ? e.oldData.mobile : e.newData.mobile,
          }
  
          console.log(newData)
          const response = await fetch(
            `https://api.recruitly.io/api/candidate?apiKey=TEST9349C0221517DA4942E39B5DF18C68CDA154`,
            {
              method: "POST",
              headers: {
                "Content-Type": "application/json",
              },
              body: JSON.stringify(newData),
            }
          );
          const responseData = await response.json();
          if (response.ok) {
            const updatedItemIndex = gridData.findIndex((item) => item.id === e.key);
            gridData.push(e.newData);
            gridData[updatedItemIndex] = e.newData;
            dataGrid.refresh();
            alert(" Record edited successfully.");
          } else {
            console.error("Failed to update record:", responseData.error);
          }
        } catch (error) {
          console.error("Failed to update record:", error);
        }
      },
      onRowRemoving: async (e) => {
        console.log("Data being sent to API:", e.data);
        try {
          const response = await fetch(
            `https://api.recruitly.io/api/candidate/${e.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`,
            {
              method: "DELETE",
            }
          );
          if (response.ok) {
            const removedItemIndex = gridData.findIndex((item) => item.id === e.key);
            if (removedItemIndex > -1) {
              gridData.splice(removedItemIndex, 1);
              dataGrid.refresh();
              alert(" Record Deleted successfully");
            }
          } else {
            console.error("Failed to delete record.");
          }
        } catch (error) {
          console.error("Failed to delete record:", error);
        }
      },
        onInitialized: () => {},
      }
    );
    fileInput = document.createElement("input");
    fileInput.type = "file";
    fileInput.accept = ".pdf,.doc,.docx";
    fileInput.addEventListener("change", handleFileSelection);

    const container = document.getElementById("fileInputContainer");
    container.appendChild(fileInput);
  });
</script>

<style>
  #dataGrid {
    height: 600px;
  }



  .popup-container button {
    display: block;
    margin-bottom: 10px;
  }
  .popup-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 9999;
  }

  .popup-content {
    background-color: white;
    padding: 20px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
    max-width: 80%;
    max-height: 80%;
    overflow: auto;
  }
</style>

<h1 style="color: blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>
{#if isFileSelectionPopupOpen}
  <div class="popup-container">
    <div class="popup-content">
      <h3>Select your file</h3>
      <input type="file" accept=".pdf,.doc,.docx" on:change={handleFileSelection} />
      <button on:click={handleFileUpload}>Upload</button>
    </div>
  </div>
{/if}
