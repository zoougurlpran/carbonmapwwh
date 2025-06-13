# carbonmapwwh 
// ... existing code up to Papa.parse complete ...

let allData = [];
let countryDataMap = {}; // { 'USA': [{...}, {...}, ...], ... }

function handleFileSelect(event) {
  const file = event.target.files[0];
  if (!file) return;

  Papa.parse(file, {
    header: true,
    dynamicTyping: true,
    complete: function(results) {
      allData = results.data;

      // Group by country ISO3
      countryDataMap = {};
      allData.forEach(row => {
        if (!row.ISO3 || !row.Emissions) return;
        if (!countryDataMap[row.ISO3]) countryDataMap[row.ISO3] = [];
        countryDataMap[row.ISO3].push(row);
      });

      drawMap(allData);
    }
  });
}

function drawMap(data) {
  // Get the most recent emission data per country for the map
  const mostRecentByCountry = {};
  data.forEach(row => {
    if (!row.ISO3 || !row.Emissions || !row.Year) return;
    if (!mostRecentByCountry[row.ISO3] || row.Year > mostRecentByCountry[row.ISO3].Year) {
      mostRecentByCountry[row.ISO3] = row;
    }
  });

  const chartData = [['Country', 'Emissions']];
  Object.values(mostRecentByCountry).forEach(row => {
    chartData.push([row.Country, Number(row.Emissions)]);
  });

  google.charts.setOnLoadCallback(function() {
    const dataTable = google.visualization.arrayToDataTable(chartData);
    const options = {
      colorAxis: {colors: ['#EFEFFF', '#02386F']},
      backgroundColor: '#81d4fa',
      datalessRegionColor: '#f8bbd0',
      defaultColor: '#f5f5f5'
    };
    const chart = new google.visualization.GeoChart(document.getElementById('map'));
    chart.draw(dataTable, options);

    // Add a 'select' event listener
    google.visualization.events.addListener(chart, 'select', function() {
      const selection = chart.getSelection();
      if (selection.length === 0) return;
      const countryName = dataTable.getValue(selection[0].row, 0);
      const iso3 = Object.values(mostRecentByCountry).find(r => r.Country === countryName).ISO3;
      showCountryDetails(iso3);
    });
  });
}

function showCountryDetails(iso3) {
  // Get all years for that country
  const rows = countryDataMap[iso3];
  if (!rows) return;

  // Sort by year ascending
  rows.sort((a,b) => a.Year - b.Year);

  // Most recent data
  const mostRecent = rows[rows.length - 1];

  // Show data and chart
  let detailsDiv = document.getElementById('country-details');
  if (!detailsDiv) {
    detailsDiv = document.createElement('div');
    detailsDiv.id = 'country-details';
    document.body.appendChild(detailsDiv);
  }
  detailsDiv.innerHTML = `<h3>${mostRecent.Country}</h3>
    <p>Most Recent Year: ${mostRecent.Year}</p>
    <p>Emissions: ${mostRecent.Emissions}</p>
    <div id="country-chart" style="width: 400px; height: 300px;"></div>`;

  // Draw the chart
  const chartData = [['Year', 'Emissions']];
  rows.forEach(row => {
    chartData.push([String(row.Year), Number(row.Emissions)]);
  });

  const dataTable = google.visualization.arrayToDataTable(chartData);
  const options = {
    title: 'Emissions Over Time',
    curveType: 'function',
    legend: { position: 'bottom' }
  };
  const chart = new google.visualization.LineChart(document.getElementById('country-chart'));
  chart.draw(dataTable, options);
}
