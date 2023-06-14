<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let isPopupVisible = false;

  function openPopup() {
    isPopupVisible = true;
  }

  function closePopup() {
    isPopupVisible = false;
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
              alert("Downloaded Successfully.");
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
                isPopupVisible = true;
                const popupContainer = document.createElement("div");
                popupContainer.className = "popup-container";

                const popupContent = document.createElement("div");
                popupContent.className = "popup-content";

                const closeButton = document.createElement("button");
                closeButton.className = "popup-close";
                closeButton.innerText = "Close";
                closeButton.addEventListener("click", () => {
                  isPopupVisible = false;
                });

                const cvContent = document.createElement("div");
                cvContent.innerHTML = cvHtml;

                popupContent.appendChild(closeButton);
                popupContent.appendChild(cvContent);
                popupContainer.appendChild(popupContent);
                document.body.appendChild(popupContainer);
              } else {
                alert("No CV data available.");
              }
            } else {
              alert("Failed to fetch CV file.");
            }
          });
          container.appendChild(viewButton);
        },
      },
    ];

    new DevExpress.ui.dxDataGrid(document.getElementById("gridContainer"), {
      dataSource: gridData,
      columns: columns,
      width: "100%",
      height: "100%",
      paging: {
        pageSize: 10,
      },
      pager: {
        visible: true,
        showPageSizeSelector: true,
        allowedPageSizes: [10, 20, 50],
      },
    });
  });
</script>

<style>
  .popup-container {
    display: flex;
    align-items: center;
    justify-content: center;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.8);
    z-index: 9999;
  }

  .popup-content {
    background-color: #fff;
    padding: 20px;
  }

  .popup-close {
    position: absolute;
    top: 10px;
    right: 10px;
    font-size: 20px;
    cursor: pointer;
  }
</style>

<main>
  <h1>Grid Example</h1>

  <div id="gridContainer"></div>

  {#if isPopupVisible}
    <div class="popup-container">
      <div class="popup-content">
        <button class="popup-close" on:click={closePopup}>Close</button>
        <div id="cvContainer"></div>
      </div>
    </div>
  {/if}
</main>
