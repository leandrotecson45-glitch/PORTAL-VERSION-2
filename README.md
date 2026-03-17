<!DOCTYPE html>
<html>
<head>

<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>PORTAL</title>

<script src="https://cdn.jsdelivr.net/npm/piexifjs"></script>

<style>

body{
margin:0;
font-family:Arial;
background:url("https://i.postimg.cc/B6X6X5Y5/1759188359397.jpg") no-repeat center center;
background-size:cover;
min-height:100vh;
display:flex;
justify-content:center;
align-items:center;
color:white;
}

body::before{
content:"";
position:fixed;
top:0;
left:0;
width:100%;
height:100%;
background:rgba(0,0,0,0.6);
z-index:-1;
}

.container{
width:100%;
max-width:520px;
background:rgba(15,23,42,0.9);
padding:20px;
border-radius:15px;
box-shadow:0 0 25px rgba(0,0,0,0.7);
}

/* 🔥 PROFESSIONAL TITLE DESIGN */
.header{
border-radius:15px;
padding:25px;
margin-bottom:15px;
text-align:center;

background:
linear-gradient(rgba(0,0,0,0.6),rgba(0,0,0,0.6)),
url("https://i.ibb.co/YTX5jq95/ow.jpg");

background-size:cover;
background-position:center;

box-shadow:0 5px 20px rgba(0,0,0,0.6);
}

.header h1{
margin:0;
font-size:50px;
color:WHITE;
text-shadow:2px 2px 8px black;
}

.header p{
margin-top:5px;
font-size:20px;
color:SKYBLUE;
text-shadow:1px 1px 5px black;
}

/* FORM */
label{
display:block;
margin-top:10px;
font-size:13px;
}

input{
width:100%;
padding:9px;
margin-top:4px;
border-radius:6px;
border:none;
background:#1e293b;
color:white;
}

.preview{
margin-top:10px;
border-radius:8px;
overflow:hidden;
border:2px dashed #475569;
}

.preview img{
width:100%;
}

button{
width:100%;
padding:12px;
margin-top:15px;
border:none;
border-radius:8px;
background:#3b82f6;
color:white;
font-size:15px;
cursor:pointer;
}

button:hover{
background:#2563eb;
}

.info{
font-size:12px;
margin-top:5px;
color:#cbd5f5;
}

</style>

</head>

<body>

<div class="container">

<!-- 🔥 HEADER DESIGN -->
<div class="header">
<h1>Ostrom Climate</h1>
<p>Action Matters</p>
</div>

<label>Upload Photo</label>
<input type="file" id="upload">

<div class="preview">
<img id="preview">
</div>

<div class="info" id="filesize"></div>
<div class="info" id="resolution"></div>

<label>File Name</label>
<input id="filename">

<label>Camera Make</label>
<input id="make">

<label>Camera Model</label>
<input id="model">

<label>Software</label>
<input id="software">

<label>Date Taken</label>
<input id="date">

<label>Latitude (6 decimal)</label>
<input id="lat">

<label>Longitude (6 decimal)</label>
<input id="lng">

<label>Image Description</label>
<input id="desc">

<label>Output Quality</label>
<input type="range" min="0.2" max="1" step="0.05" value="0.9" id="quality">

<button onclick="downloadImage()">Download Edited Image</button>

</div>

<script>

let imageData=""

document.getElementById("upload").addEventListener("change",function(e){

const file=e.target.files[0]

document.getElementById("filename").value=file.name.replace(/\.[^/.]+$/,"")

document.getElementById("filesize").innerHTML=
"File Size: "+(file.size/1024).toFixed(2)+" KB"

const reader=new FileReader()

reader.onload=function(event){

imageData=event.target.result
document.getElementById("preview").src=imageData

const img=new Image()

img.onload=function(){
document.getElementById("resolution").innerHTML=
"Resolution: "+img.width+" x "+img.height
}

img.src=imageData

loadExif(imageData)

}

reader.readAsDataURL(file)

})

function dmsToDecimal(dms,ref){

let degrees=dms[0][0]/dms[0][1]
let minutes=dms[1][0]/dms[1][1]
let seconds=dms[2][0]/dms[2][1]

let decimal=degrees+(minutes/60)+(seconds/3600)

if(ref==="S"||ref==="W"){
decimal=-decimal
}

return decimal.toFixed(6)

}

function degToDmsRational(deg){

let abs=Math.abs(deg)

let d=Math.floor(abs)
let m=Math.floor((abs-d)*60)
let s=((abs-d)*60-m)*60

return [
[d,1],
[m,1],
[Math.round(s*1000000),1000000]
]

}

function loadExif(data){

try{

const exif=piexif.load(data)

document.getElementById("make").value=
exif["0th"][piexif.ImageIFD.Make]||""

document.getElementById("model").value=
exif["0th"][piexif.ImageIFD.Model]||""

document.getElementById("software").value=
exif["0th"][piexif.ImageIFD.Software]||""

document.getElementById("desc").value=
exif["0th"][piexif.ImageIFD.ImageDescription]||""

document.getElementById("date").value=
exif["Exif"][piexif.ExifIFD.DateTimeOriginal]||""

if(exif["GPS"][piexif.GPSIFD.GPSLatitude]){

let lat=dmsToDecimal(
exif["GPS"][piexif.GPSIFD.GPSLatitude],
exif["GPS"][piexif.GPSIFD.GPSLatitudeRef]
)

let lng=dmsToDecimal(
exif["GPS"][piexif.GPSIFD.GPSLongitude],
exif["GPS"][piexif.GPSIFD.GPSLongitudeRef]
)

document.getElementById("lat").value=lat
document.getElementById("lng").value=lng

}

}catch(e){

console.log("No EXIF metadata")

}

}

function downloadImage(){

const make=document.getElementById("make").value
const model=document.getElementById("model").value
const software=document.getElementById("software").value
const date=document.getElementById("date").value
const lat=parseFloat(document.getElementById("lat").value)
const lng=parseFloat(document.getElementById("lng").value)
const desc=document.getElementById("desc").value
const filename=document.getElementById("filename").value
const quality=parseFloat(document.getElementById("quality").value)

let zeroth={}
let exif={}
let gps={}

zeroth[piexif.ImageIFD.Make]=make
zeroth[piexif.ImageIFD.Model]=model
zeroth[piexif.ImageIFD.Software]=software
zeroth[piexif.ImageIFD.ImageDescription]=desc

exif[piexif.ExifIFD.DateTimeOriginal]=date

if(!isNaN(lat)&&!isNaN(lng)){

gps[piexif.GPSIFD.GPSLatitudeRef]=lat>=0?"N":"S"
gps[piexif.GPSIFD.GPSLatitude]=degToDmsRational(lat)

gps[piexif.GPSIFD.GPSLongitudeRef]=lng>=0?"E":"W"
gps[piexif.GPSIFD.GPSLongitude]=degToDmsRational(lng)

}

const exifObj={"0th":zeroth,"Exif":exif,"GPS":gps}

const exifBytes=piexif.dump(exifObj)

let img=new Image()

img.onload=function(){

let canvas=document.createElement("canvas")
canvas.width=img.width
canvas.height=img.height

let ctx=canvas.getContext("2d")
ctx.drawImage(img,0,0)

let compressed=canvas.toDataURL("image/jpeg",quality)

let finalImage=piexif.insert(exifBytes,compressed)

let link=document.createElement("a")
link.href=finalImage
link.download=filename+".jpg"
link.click()

}

img.src=imageData

}

</script>

</body>
</html>
