<div id='companies' class='row'>
	
		<a class='item col-md' href='/index.php'>
			<div class='icon'></div>
			<div class='title'>Автомобильный завод "НАЗ"</div>
			<div class='text'>Российский производитель автомобилей LCV, расположенный в Нижнем Новгороде.
			</div>
		</a>
		
		<a class='item col-md' href='/index.php'>
			<div class='icon'></div>
			<div class='title'>Нижегородские автокомпоненты </div>
			<div class='text'>Российский производитель автокомпонентов ГАЗ, расположенный в Нижнем Новгороде
			</div>
		</a>		
	</div>		

 #companies{
	margin:30px 0;
	justify-content: space-between;	
}

#companies .item{	
	display:block;
    text-transform: none;
    font-weight: 0;
    font-size: unset;
    line-height: unset;	
	
	float: left;
	background: #ffffff;
	width:268px;
	margin-right: 26px;
	
	margin-bottom: 26px;
	overflow: hidden;
	padding:15px;
	
}

#companies .item:nth-child(4n){
	margin-right: 0;
}

#companies .item .icon{
	height:75px;
	background-image:url( ./img/company.png );
	background-size:contain;
	background-repeat: no-repeat;
	background-position:left;
}

#companies .item .title{
	margin:10px 0;
	height:50px;    
    font-weight: 600;
	font-size: 20px;	
	color: #000000;
}

#companies .item .text{
	margin-bottom:20px;
	height:160px;	
	font-size: 18px;	
}
