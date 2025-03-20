# Data-Visualization-P5
Data Visualization attempt on P5.js by loading data from Json File

I wanted to create linear plot of countries where nobel-prize laureates were born. Based on the data, I made a plot of top 25 countries that have produced the most nobel-prize laureates. Data was taken from : https://api.nobelprize.org/v1/laureate.json

[P5.js Sketch Link](https://editor.p5js.org/map.gamedev.ubi/sketches/TjjnKYgQy)

## Code:
```
let laureates_data_arr = [];
let country_data_arr = [];
let points = [];

let line_padding_left = 50;
let line_padding_bottom = 50;

let countries_to_display = 20;
let distance_between_points = 30;
let point_size = 7.5;

function preload(){
  laureate_json_data = loadJSON('laureate.json');
}

function setup() {
  createCanvas(700, 500);
  init_data();
  draw_sketch();
}

function draw_sketch() {
  background(255);
  
  stroke(0);
  draw_y_axis();
  draw_axis_lines();
  draw_country_code_name();
  
  draw_points();
  draw_connector_lines();
}

function draw_axis_lines(){
  strokeWeight(2);
  line(line_padding_left, line_padding_bottom, line_padding_left, height-line_padding_bottom);
  line(line_padding_left, height-line_padding_bottom, width - line_padding_left / 2, height-line_padding_bottom);
}

function draw_points(){
  textSize(12)
  stroke(0);
  fill(0);
  let country_data_index;
  for(let i = 0; i < points.length; i++){
    strokeWeight(1);
    points[i].drawPoint();
    country_data_index = points[i].country_index;
    let country_code = country_data_arr[country_data_index].code;
  
    strokeWeight(1);
    let text_x_coord = points[i].x - 5;
    let text_y_coord = height - (line_padding_bottom / 2);
    text(country_code, text_x_coord, text_y_coord);
  }
}

function draw_country_code_name(){
  stroke(255);
  strokeWeight(2.5)
  textSize(12)
  let y_position_delta = 18;
  let y_position = line_padding_bottom * 0.6;
  for(let i = 0; i < countries_to_display; i++){
    let x_coord = width-225;
    let country_code = country_data_arr[i].code;
    let country_name = country_data_arr[i].name;
    let country_string = country_code + "   -   " + country_name;
    text(country_string, x_coord, y_position);
    y_position += y_position_delta;
  }
}

function draw_y_axis(){
  // strokeWeight(1)
  textSize(12)
  let last_y_coord = 0;
  let x_coord = line_padding_left / 3;
  for(let i = points.length - 1; i > -1; i--){
    let y_coord = points[i].y;
    if(abs(y_coord - last_y_coord) > 20){
      last_y_coord = y_coord;
      let country_index = points[i].country_index;
      let country_count = country_data_arr[country_index].count;
      text(country_count, x_coord, last_y_coord+4);
    }
  }
}

function draw_connector_lines(){
  strokeWeight(1);
  for(let i = 1; i < points.length; i++){
    let first_point = points[i-1];
    let second_point = points[i];
    line(first_point.x, first_point.y, second_point.x, second_point.y);
  }
}


function sort_country_data(){
  for(let i = 0; i < country_data_arr.length - 1; i++){
    for(let j = i+1; j < country_data_arr.length; j++){
      if(country_data_arr[i].count < country_data_arr[j].count){
        let temp = country_data_arr[i];
        country_data_arr[i] = country_data_arr[j];
        country_data_arr[j] = temp;
      }
    }
  }
}

function init_data(){
  load_json_data(laureate_json_data);
  populate_country_data();
  sort_country_data();
  init_point_data();
}

function init_point_data(){
  for(let i = 0; i < countries_to_display; i++){
    let x_coord = ((i+1) * distance_between_points) + line_padding_left;
    let y_coord = (height-line_padding_bottom) - country_data_arr[i].count - 20;
    points.push(new PointData(x_coord, y_coord, i));
  }
}

function populate_country_data(){
  if(laureates_data_arr.length == 0){
    return;
  }
  
  for(let j = 0; j < laureates_data_arr.length; j++){
    let country_code = laureates_data_arr[j].born_country_code;
    let country_name = laureates_data_arr[j].born_country_name;
    
    let country_data_index = -1;
    
    for(let i = 0; i < country_data_arr.length; i++){
      if(country_data_arr[i].code == country_code){
        country_data_index = i;
        break;
      }
    }
    
    if(country_data_index == -1){
      country_data_arr.push(new CountryData(country_code, country_name));
    }
    else{
      country_data_arr[country_data_index].update_count();
    }
  }
}

function load_json_data(json_data){
  let json_keys_arr = Object.entries(json_data);
  let first_name_key = "firstname", last_name_key = "surname", country_code = "bornCountryCode", country_name = "bornCountry";
  
  for(let json_key in json_keys_arr){
    let json_obj = json_data[json_key];
    let fill_count = 0;
    
    for(let json_obj_key in json_obj){
      if(json_obj_key == first_name_key || json_obj_key == last_name_key || json_obj_key == country_code){
        fill_count++;
      }
    }
    
    if(fill_count == 3){
      laureates_data_arr.push(new LaureateData(json_obj[first_name_key], json_obj[last_name_key], json_obj[country_code], json_obj[country_name]));
    }
  }
}

class PointData{
  constructor(x, y, country_data_index){
    this.x = x;
    this.y = y;
    this.country_index = country_data_index;
  }
  
  drawPoint(){
    circle(this.x, this.y, point_size);
  }
}

class CountryData{
  constructor(country_code, country_name){
    this.code = country_code;
    this.name = country_name;
    this.count = 1;
  }
  
  update_count(){
    this.count++;
  }
}

class LaureateData{
  constructor(first_name, last_name, born_country_code, born_country_name){
    this.first_name = first_name;
    this.last_name = last_name;
    this.born_country_code = born_country_code;
    this.born_country_name = born_country_name;
  }
}

```
