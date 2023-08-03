<script>
  import { getContext} from "svelte";

  export let text;
  export let dataProvider;
  export let filename;

  const { styleable } = getContext("sdk");
  const component = getContext("component");

  function objectToCsvRow(obj) {
    const values = Object.values(obj);
    const escapedValues = values.map((value) => {
      // If the value contains a comma or a double-quote, wrap it in double-quotes and escape existing double-quotes
      // if (value.includes(",") || value.includes('"')) {
      //   return `"${value.replace(/"/g, '""')}"`;
      // }
      return value;
    });
    return escapedValues.join(",");
  }

  // Function to generate the CSV string
  function generateCsvString(data) {
    const headers = Object.keys(data[0]).filter(
      (key) => !excludedColumns.includes(key)
    );
    const csvRows = data.map((obj) => objectToCsvRow(obj, excludedColumns));

    // Join headers and rows with line breaks to form the final CSV string
    return [headers.join(","), ...csvRows].join("\n");
  }
  const excludedColumns = ["_id", "_rev"];

  // Function to handle the button click event
  function saveDataToFile() {
    let data = dataProvider?.rows || [];
    // Create a Blob (Binary Large Object) from the data
    if (data) {
      const blob = new Blob([generateCsvString(data)], { type: "text/plain" });

      // Create a temporary URL for the blob
      const url = URL.createObjectURL(blob);

      // Create a link element
      const link = document.createElement("a");

      // Set the link's attributes
      link.href = url;
      if (!filename) {
        filename = "iot-erp-bridge-datenexport";
      }
      link.download = filename + ".csv"; // Set the file name

      // Append the link to the DOM (necessary for Firefox)
      document.body.appendChild(link);

      // Programmatically trigger a click event on the link to start the download
      link.click();

      // Clean up by removing the link and revoking the URL
      link.remove();
      URL.revokeObjectURL(url);
    }
  }

</script>

<div use:styleable={$component.styles}>
  <div>
    <button on:click={saveDataToFile}>{text}</button>
  </div>
</div>

<style>
  /* You can style the button as you like */
  button {
    padding: 10px 20px;
    background-color: #007bff;
    color: #fff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }
</style>
