<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<title>Задание</title>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="styles.css" />
</head>

<body>

<form name="formCountry" id="firstForm" method="post" >

<?php
$link = mysqli_connect("localhost","user","123456", "Test");//подключение к базе

//проверка установленного соединения
if ($link==NULL) {
    echo "Не удалось подключиться к MySQL: (" .$mysqli->connect_errno. ") ".mysqli_connect_error();
}

if(isset($_POST['setcountry'])) //апдейты базы при отправке формы
{
	$_POST['setcountry'] = htmlspecialchars($_POST['setcountry'], ENT_QUOTES,"UTF-8");

	if($_POST['ondelete']) //при удалении
		$Y = mysqli_query($link,"DELETE FROM Countries WHERE country = '".$_POST['setcountry']."'");
	else
	$Y = mysqli_query($link,"INSERT INTO Countries( country ) VALUES ('".$_POST['setcountry']."')");

}
else if(isset($_POST['setcity']))
{
	//echo "INSERT INTO Cities (cityName, countryId) VALUES ( '".$_POST['setcity']."',".$_POST['myCountries'].")";
	$_POST['setcity'] = htmlspecialchars($_POST['setcity'], ENT_QUOTES,"UTF-8");
	
	if($_POST['ondelete'])//при удалении
		$Y = mysqli_query($link,"DELETE FROM Cities WHERE cityName = '".$_POST['setcity']."'");
	else
$Y = mysqli_query($link,"INSERT INTO Cities (cityName, countryId) VALUES ( '".$_POST['setcity']."',".$_POST['myCountries'].")");
     
}
else if(isset($_POST['setlan']))
{
	$_POST['setlan'] = htmlspecialchars($_POST['setlan'],ENT_QUOTES,"UTF-8");
	//echo var_dump($_POST);
	$counter = 0;

	$res = mysqli_query($link,"SELECT IdCities FROM Languages WHERE language = '".$_POST['setlan']."'");
	if(mysqli_num_rows($res)!=0 && !$_POST['ondelete']) //при добавлении
	{
		$row = mysqli_fetch_assoc($res);
		$el = explode("," , $row['IdCities']);//получение id городов языка
		//echo var_dump($el);
		for($i=0;$i<count($el);$i++)
		{
			if($_POST['val1'] == $el[$i])//проверка на наличие id этого города
			{
				$counter = 1;
				break;
			}
		}

		if($counter == 0)//если города еще нет в списке
		{
		$com = ",";
		if($row['IdCities']=="" || $row['IdCities']==",") //убираем разделитель на случай если нет внесённых значений
			$com = "";	
			
			$row['IdCities'] .= $com.$_POST['val1']; //если нет в списке добавляем id городаs
		$res = mysqli_query($link,"UPDATE Languages SET IdCities = '".$row['IdCities']."' WHERE language = '".$_POST['setlan']."'");
		
		}
	}
	else  if(mysqli_num_rows($res)==0)//Если такого языка еще нет в списке языков
	{
	mysqli_query($link,"INSERT INTO Languages ( IdCities, language ) VALUES ('".$_POST['val1']."','".$_POST['setlan']."')");

		//echo "INSERT INTO Languages ( IdCities, language ) VALUES ('".$_POST['val1']."','".$_POST['setlan']."')";
	}

	if($_POST['ondelete'] && mysqli_num_rows($res)!=0) //при удалении языка города
	{
		$row = mysqli_fetch_assoc($res);
		$row['IdCities'] = preg_replace("/[".$_POST['val1']."],?/", "",$row['IdCities']); //заменяет идентификатор города
	//обновляет список ид городов
		echo $row['IdCities'];
	mysqli_query($link,"UPDATE Languages SET IdCities = '".$row['IdCities']."' WHERE language = '".$_POST['setlan']."'");
	}
	
}

$res = mysqli_query($link,"SELECT * FROM Countries");//запрос из таблицы стран

echo "<div class='linemeny'><select name='myCountries' id='sel1' onchange='change()' >";//формирует список стран
//обработка результатов запроса
 while( $row = mysqli_fetch_assoc($res)) {
        //printf("%s (%s)\n", $row['id'], $row['country']);
	echo "<option value=".$row['id'].">".$row['country']."</option>";

}
echo "</select></div>";
mysqli_free_result($res);//Освобождает используемую память
$res = mysqli_query($link,"SELECT * FROM Cities");//запрос городов

echo "<div class='linemeny'><select name='myCities' id='sel2' onchange='change1()' >";//формирует список городов
while($row = mysqli_fetch_assoc($res)) {
	echo "<option id=".$row['id']." value=".$row['countryId'].">".$row['cityName']."</option>";
}
echo "</select></div>";

mysqli_free_result($res);//Освобождает используемую память
$res = mysqli_query($link,"SELECT IdCities, language FROM Languages");//запрос языков
echo "<div class='linemeny'><select name='myLang' id='sel3' >";//формирует список языков
while($row = mysqli_fetch_assoc($res)) {
	echo "<option value=".$row['IdCities'].">".$row['language']."</option>";
}
echo "</select></div>";

mysqli_free_result($res);//Освобождает используемую память
mysqli_close($link);//Закрывает соединение

echo "\n";
?>

<div class="cage">.</div>
<div class="line" id="CountrList" ></div><div class="line" id="CityList"></div><div class="line" id="lanList"></div>
<div class="cage">.</div>

<div>
<p>Добавить</p><p>
<input type="radio" name="checker" id="r1" onchange="setname()" value="Страну" checked />Страну<br>
<input type="radio" name="checker" id="r2" onchange="setname()" value="Город" />Город<br>
<input type="radio" name="checker" id="r3" onchange="setname()" value="Язык" />Язык<br>
</p>

<input type="text" name="settry" id="message" /> <input type="submit" id="sub" value = "добавить" />
<input type="checkbox" id="delete" onchange="del()" />Удалить
<input type="hidden" id="firstval" name="val1" />
<input type="hidden" id="secondval" name="val2" />
</div>

<script type='text/javascript'>

var country;
var city;
var city2;
var language;
var countries;
var cityes;
var languages;

function del(){
var obj = document.getElementById("delete");
if(obj.checked != false)
{
	obj.name= "ondelete";
	document.getElementById("sub").value = "Удалить";
}
else
{
	obj.name= "nodel";
	document.getElementById("sub").value = "Добавить";
}
}

function setname(){
var ip = document.getElementsByName('checker');
var mes = document.getElementById('message');
var objSelt = document.getElementById("sel2");

	for (var i = 0; i < ip.length; i++)
	{
        	if(ip[i].checked && ip[i].id == "r1")
		{
			mes.name = "setcountry";
        	}
		else if(ip[i].checked && ip[i].id == "r2")
		{
			mes.name = "setcity";
			//document.getElementById('firstval').value = objSel.options[objSel.selectedIndex].value;
		}
		else if(ip[i].checked && ip[i].id == "r3")
		{
			mes.name = "setlan";
			//получение id выделенного города
			document.getElementById('firstval').value = city2[objSelt.options[objSelt.selectedIndex].text];
			//alert(objSelt.options[objSelt.selectedIndex].text);
		}
    	}
}

function change(){
var objSel = document.getElementById("sel1");
var objSel1 = document.getElementById("sel2");
//countries.innerHTML = "";
cityes.innerHTML = "";

objSel1.options.length = 0;//очищаем список городов
for(var p in city){

	if(objSel.selectedIndex != -1)
	{
		if(city[p] == objSel.options[objSel.selectedIndex].value)
		{
			objSel1.options[objSel1.options.length] = new Option(p,city[p]);
			objSel1.options[objSel1.options.length-1].id = city2[p];//добавляет ид города
			
			cityes.innerHTML = cityes.innerHTML +"<br />"+ p;//добавляет названия городов в таблицу
		}
		
	}	

}
//countries.innerHTML = countries.innerHTML +"<br />"+ objSel.options[objSel.selectedIndex].text;

//objSel.selectedIndex = 0;
change1(); //зависимый список языков
}

function change1(){
var objSel1 = document.getElementById("sel2");
var objSel2 = document.getElementById("sel3");

languages.innerHTML = "";
objSel2.options.length = 0;
for(var p in language){

	if(objSel1.selectedIndex != -1)
	{
		//language[p] айдишники городов в которых применяеться этот язык p
		//alert(language[p]);
		//if(language[p].split(",").indexOf(city2[p]))
		//var i = 0;
		
			if(language[p].indexOf(Number(objSel1.options[objSel1.selectedIndex].id)) != -1) //нахождение id города в списке языков
			{
				languages.innerHTML = languages.innerHTML +"<br />"+ p;
				objSel2.options[objSel2.options.length] = new Option(p,language[p]);
			}
		
	}
}
//objSel1.selectedIndex = 0;
objSel2.selectedIndex = 0;
setname();
}

function firstId(){
var objSel = document.getElementById("sel1");
var objSel1 = document.getElementById("sel2");
var objSel2 = document.getElementById("sel3");
countries = document.getElementById("CountrList");
cityes = document.getElementById("CityList");
languages = document.getElementById("lanList");


country = new Object(); // страны
city = new Object();    // города и id стран
language = new Object();// языки и id городов
city2 = new Object(); // города и id языков

for(var i = 0; i < objSel.options.length ; i++)
{
	country[objSel.options[i].value] = objSel.options[i].text;
	countries.innerHTML = countries.innerHTML +"<br />"+ objSel.options[i].text;
}

for(var i = 0; i < objSel1.options.length ; i++)
{
	city[objSel1.options[i].text] = objSel1.options[i].value;
	city2[objSel1.options[i].text] = objSel1.options[i].id; //отедльно заносим индексы городов
	cityes.innerHTML = cityes.innerHTML +"<br />"+ objSel1.options[i].text;

}
for(var i = 0; i < objSel2.options.length ; i++)
{
	language[objSel2.options[i].text] = objSel2.options[i].value;//.split(",");
	languages.innerHTML = languages.innerHTML +"<br />"+ objSel2.options[i].text;
}

objSel.options[0].selected = true; //выделяем первый элемент
change();//и формируем действующий список городов
setname();
}

document.addEventListener("DOMContentLoaded", firstId);

</script>

</form>


</body>

</html>
