<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];

  onMount(async () => {
    const fetchCandidateData = async () => {
      const response = await fetch(
        `https://api.recruitly.io/api/candidatecv/${data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
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

          onInitialized: () => {
            // Additional initialization code, if needed
          },
        }
      );

      // Download file
      const downloadFile = async (cvid) => {
        const downloadUrl = `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${cvid}&apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`;
        const downloadResponse = await fetch(downloadUrl);
        const fileBlob = await downloadResponse.blob();

        const downloadLink = document.createElement("a");
        downloadLink.href = URL.createObjectURL(fileBlob);
        downloadLink.download = "file.pdf"; // Replace "file.pdf" with the desired file name
        downloadLink.click();
      };

      // Call the downloadFile function with the desired cvid
      downloadFile(cvid);
    };

    await fetchCandidateData();
  });
</script>

<style>
  #dataGrid {
    height: 400px;
  }
</style>

<h1 style="color:blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>
