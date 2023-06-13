<script>
	import { onMount } from "svelte";
	import "bootstrap/dist/css/bootstrap.min.css";
	import DevExpress from "devextreme";
  
	let jsonData = [];
	let gridData = [];
  
	async function fetchCandidateData(candidateId) {
  const apiUrl = `https://api.recruitly.io/api/candidatecv/${candidateId}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`;

  const response = await fetch(apiUrl, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(options.data), // Assuming options.data is the data object to be sent
  });

  const data = await response.json();

  return data;
}


  
	async function downloadFile(id) {
	  const downloadUrl = `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${id}&apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`;
  
	  // Fetch the file
	  const response = await fetch(downloadUrl);
  
	  // Convert the file response to a blob
	  const fileBlob = await response.blob();
  
	  // Create a temporary anchor element to initiate the download
	  const link = document.createElement("a");
	  link.href = URL.createObjectURL(fileBlob);
	  link.download = `file-${id}.txt`; // Specify the file name here
	  link.click();
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
		  caption: "Download CV",
		  width: 150,
		  cellTemplate: function (container, options) {
			const downloadButton = document.createElement("button");
			downloadButton.innerText = "Download";
			downloadButton.addEventListener("click", function () {
			  downloadFile(options.data.cvid);
			});
			container.appendChild(downloadButton);
		  },
		},
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
	});
</script>

<style>
	#dataGrid {
	  height: 400px;
	}
</style>

<h1 style="color:blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>
