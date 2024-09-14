デイリーノートに書く

[収入::内訳:金額]
[支出::内訳:金額]

```dataviewjs
const today = new Date();
let curMonth = today.getMonth() + 1;
let befMonth = today.getMonth() + 0;
if(curMonth < 10)　curMonth = '0' + curMonth;
if(befMonth < 10)　befMonth = '0' + befMonth;

//------------------------------------------------------------
const column = ["日付", "内訳", "収入", "支出", "残金"];

let table = [];

let curTag = "#仕訳/2024/" + curMonth;
let befTag = "#仕訳/2024/" + befMonth;

let curFilesWithTag = dv.pages(curTag);
let curFiles = Array.from(curFilesWithTag);

let befFilesWithTag = dv.pages(befTag);
let befFiles = Array.from(befFilesWithTag);

let zankin = 0;
let shunyuResult = 0;
let sishutuResult = 0;

//render
dv.paragraph("**使用しているタグ** ： " + curTag);
createTable();

//------------------------------------------------------------

function calcKurikoshi(){
	for(let file of befFiles){
		getShunyu(file, false);
		getSishutu(file, false);
	}
	
	addListItem(true, "繰越", befMonth + "月からの繰越", zankin, 0, 0);
}

//-----------------------------------------------------------

function createTable(){
	calcKurikoshi();
	
	shunyuResult = zankin;
	sishutuResult = 0;

	for(let file of curFiles){
		getShunyu(file, true);
		getSishutu(file, true);
	}
	
	addListItem(true, "", "");
	addListItem(
		true, 
		"###### 合計", 
		"", 
		"###### " + shunyuResult, 
		"###### " + sishutuResult, 
		"###### " + zankin
	);
	
	dv.table(column, table);
}

function getShunyu(file, isRender){
	if(typeof file.収入 === "undefined"){
		return;
	}
	else if(file.収入.constructor === String){
		let item = file.収入.split(":");
		calcZankin("shunyu", item[1]);
		
		addListItem(
			isRender,
			convertLinkedFile(file.file.name),
			item[0],
			item[1],
			0,
			zankin
		);
		return;
	}

	for(let shunyu of file.収入){
		let item = shunyu.split(":");
		item = combineUniData(item);
		calcZankin("shunyu", item[1]);
		
		addListItem(
			isRender,
			convertLinkedFile(file.file.name),
			item[0],
			item[1],
			0,
			zankin
		);
	}
}

function getSishutu(file, isRender){
	if(typeof file.支出 === "undefined"){
		return;
	}
	else if(file.支出.constructor === String){
		let item = file.支出.split(":");
		calcZankin("sishutu", item[1]);
		
		addListItem(
			isRender,
			convertLinkedFile(file.file.name),
			item[0], 
			0, 
			item[1], 
			zankin
		 );
		return;
	}
	
	for(let sishutu of file.支出){
		let item = sishutu.split(":");
		item = combineUniData(item);
		calcZankin("sishutu", item[1]);
		
		addListItem(
			isRender,
			convertLinkedFile(file.file.name), 
			item[0], 
			0, 
			item[1], 
			zankin
		);
	}
}

function combineUniData(item){
	if(item[0].length !== 1) return item;
	return item.join();
}

function calcZankin(mode, value){
	let intValue = parseInt(value);
	if(mode == "shunyu"){
		zankin += intValue;
		shunyuResult += intValue;
	}
	else{
		zankin -= intValue;
		sishutuResult += intValue;
	}
}

function addListItem(isRender, fileName, utiwake, shunyu, sishutu, zankin){
	if(!isRender) return;
	let item = [fileName, utiwake, shunyu, sishutu, zankin];
	table.push(item);
}

function convertLinkedFile(fileName){
	return "[[" + fileName + "]]";
}
```
