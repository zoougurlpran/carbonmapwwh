<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Carbon Emissions by Country</title>
  <script src="https://d3js.org/d3.v7.min.js"></script>
  <script src="https://d3js.org/topojson.v3.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: Arial, sans-serif; margin: 0; }
    #map { width: 100%; height: 600px; }
    #chart-container { width: 80%; margin: 20px auto; }
    canvas { background: #fff; border: 1px solid #ccc; padding: 10px; }
  </style>
</head>
<body>
  <h1 style="text-align:center;">탄소 배출량 추이 시각화</h1>
  <div id="map"></div>
  <div id="chart-container">
    <canvas id="emissionChart"></canvas>
  </div>  <script>
    // 데이터 로드 (변환된 CSV 파일을 동일 디렉토리에 넣어주세요)
    Promise.all([
      d3.json("https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json"),
      d3.csv("탄소배출량_변환.csv") // 파일명은 변환 후 저장한 이름
    ]).then(([worldData, csvData]) => {
      const countryData = {};
      csvData.forEach(row => {
        const name = row["Country"];
        countryData[name] = [];
        for (let year in row) {
          if (year !== "Country") {
            countryData[name].push({ year: +year, value: +row[year] });
          }
        }
      });

      const width = 960, height = 600;
      const svg = d3.select("#map").append("svg")
        .attr("width", width)
        .attr("height", height);

      const projection = d3.geoNaturalEarth1().scale(160).translate([width / 2, height / 2]);
      const path = d3.geoPath().projection(projection);

      const countries = topojson.feature(worldData, worldData.objects.countries);

      svg.selectAll("path")
        .data(countries.features)
        .enter()
        .append("path")
        .attr("d", path)
        .attr("fill", "#ccc")
        .attr("stroke", "#333")
        .on("click", (event, d) => {
          const name = d.properties.name;
          if (countryData[name]) {
            drawChart(name, countryData
