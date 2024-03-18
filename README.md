# RelativityOne-Bookmarklets
A collection of bookmarklets for Relativity(One) administrators.


**Bookmarklets:**
- [Show Audit](#show-audit)
- [Edit Longtext Field](#edit-longtext-field)
- [Stop Inactivity Logout](#stop-inactivity-logout)
- [Republish Processing Sets](#republish-processing-sets)
- [Select All Searches](#select-all-searches)
- [Export Saved Searches](#export-saved-searches)
- [Filter Analytics Entities From All Custodians](#filter-analytics-entities-from-all-custodians)
- [Download Native File (ECA workspace)](#download-native-file)
- [Show Active Learning Projects (ECA workspace)](#show-active-learning-projects)
- [Show Batch Sets (ECA workspace)](#show-batch-sets)


### Show Audit
Opens a new window with an Audit Log of the currently displayed object. Useful for tracking down changes made to Saved Searches, Folders, or similar objects. Simply run this bookmarklet while having the results of a saved search, folder, or any other object loaded.
```js
let host = window.top.location.host;
let urlparams = new URLSearchParams(window.top.location.search);
let workspaceid = urlparams.get('AppID');
let artifactid = urlparams.get('SelectedSearchArtifactID');
if (artifactid === null) {
	artifactid = urlparams.get('SelectedFolderArtifactID');
	if (artifactid === null) {
		artifactid = urlparams.get('DocumentID');
		if (artifactid === null) {
			artifactid = urlparams.get('ArtifactID');
			if (artifactid === null) {
				urlparams = new URLSearchParams(urlparams.get('directto'));
				artifactid = urlparams.get('ArtifactID');
			}
		}
	}
}
if (artifactid === null) {
	alert('No ArtifactID field found');
} else {
	window.open('https://' + host + '/Relativity//Audit.aspx?AppID=' + workspaceid.toString() + '&ArtifactID=' + artifactid.toString(), '_blank', 'popup');
}
```
To install this bookmarklet, create a new entry in your browser's Favorites Bar and set its URL to the following string:
```js
javascript:(function()%7Blet%20host%20%3D%20window.top.location.host%3Blet%20urlparams%20%3D%20new%20URLSearchParams(window.top.location.search)%3Blet%20workspaceid%20%3D%20urlparams.get('AppID')%3Blet%20artifactid%20%3D%20urlparams.get('SelectedSearchArtifactID')%3Bif%20(artifactid%20%3D%3D%3D%20null)%20%7Bartifactid%20%3D%20urlparams.get('SelectedFolderArtifactID')%3Bif%20(artifactid%20%3D%3D%3D%20null)%20%7Bartifactid%20%3D%20urlparams.get('DocumentID')%3Bif%20(artifactid%20%3D%3D%3D%20null)%20%7Bartifactid%20%3D%20urlparams.get('ArtifactID')%3Bif%20(artifactid%20%3D%3D%3D%20null)%20%7Burlparams%20%3D%20new%20URLSearchParams(urlparams.get('directto'))%3Bartifactid%20%3D%20urlparams.get('ArtifactID')%3B%7D%7D%7D%7Dif%20(artifactid%20%3D%3D%3D%20null)%20%7Balert('No%20ArtifactID%20field%20found')%3B%7D%20else%20%7Bwindow.open('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity%2F%2FAudit.aspx%3FAppID%3D'%20%2B%20workspaceid.toString()%20%2B%20'%26ArtifactID%3D'%20%2B%20artifactid.toString()%2C%20'_blank'%2C%20'popup')%3B%7D%7D)()
```

### Edit Longtext Field
There are situations when you need to make manual adjustments to Extracted Text, OCR Text, or Transcribed Text of a specific document. This bookmarklet lets you edit any currently displayed longtext field straight from the document's Text Viewer without going through the hassle of creating an Import/Export task.
```js
let csrftoken = window.top.GetCsrfTokenFromPage();
let host = window.top.location.host;
let urlparams = new URLSearchParams(window.top.location.search);
let workspaceid = urlparams.get('AppID');
let documentid = urlparams.get('DocumentID');
let reviewinterface = document.getElementById('_ReviewInterface').contentWindow.document;
let fieldname = reviewinterface.getElementById('ctrl-ri-tab-text-viewer').children[0].children[0].children[0].children[0].innerText;

if (urlparams.get('ViewerType') == 'text') {
	fetch('https://' + host + '/Relativity.REST/api/Relativity-DocumentViewer/v1/PrivateDocumentViewerServiceManager/GetReviewInterfaceData', {
		'headers': {
			'accept': '*/*',
			'accept-language': 'en-US,en;q=0.9',
			'content-type': 'application/json',
			'sec-fetch-dest': 'empty',
			'sec-fetch-mode': 'cors',
			'sec-fetch-site': 'same-origin',
			'x-csrf-header': csrftoken
		},
		'referrerPolicy': 'strict-origin-when-cross-origin',
		'body': '{"request":{"WorkspaceId":' + workspaceid.toString() + ',"ViewerPreference":7,"Options":{"ClientId":"Relativity.ReviewInterface.PrepareDocuments"},"DocumentIds":[' + documentid.toString() + ']}}',
		'method': 'POST',
		'mode': 'cors',
		'credentials': 'include'
	}).then(response => {
		return response.json();
	}).then(jsn => {
		let LongTextFields = Object.values(jsn['DocumentData'])[0]['LongTextVisibility'];
		let longtextfieldid;
		for (const id in LongTextFields) {
			if (LongTextFields[id]['TextFieldName'] == fieldname) {
				longtextfieldid = LongTextFields[id]['FieldArtifactId'];
			}
		}

		fetch('https://' + host + '/Relativity.REST/api/Relativity-DocumentViewer/v1/PrivateDocumentViewerServiceManager/GetTextViewerContent', {
			'headers': {
				'accept': '*/*',
				'accept-language': 'en-US,en;q=0.9',
				'content-type': 'application/json',
				'sec-fetch-dest': 'empty',
				'sec-fetch-mode': 'cors',
				'sec-fetch-site': 'same-origin',
				'x-csrf-header': csrftoken
			},
			'referrerPolicy': 'strict-origin-when-cross-origin',
			'body': '{"request":{"WorkspaceId":' + workspaceid.toString() + ',"ArtifactId":' + documentid.toString() + ',"LongTextFieldArtifactId":' + longtextfieldid.toString() + ',"PageSize":999999999,"PageIndex":0}}',
			'method': 'POST',
			'mode': 'cors',
			'credentials': 'include'
		}).then(response => {
			return response.json();
		}).then(jsn => {
			let oitpageblock;
			for (const oit in reviewinterface.getElementsByClassName('ri-text-viewer')[0].getElementsByClassName('oit-pageblock')) {	
				if (reviewinterface.getElementsByClassName('oit-pageblock')[oit].className == 'oit-pageblock') {
					oitpageblock = reviewinterface.getElementsByClassName('ri-text-viewer')[0].getElementsByClassName('oit-pageblock')[oit].parentNode;
					break;
				}
			}
			let originalcontent = oitpageblock.innerHTML;
			oitpageblock.style.textAlign = 'center';
			oitpageblock.innerHTML = '<div class="oit-pageblock"><button id="cancelbutton" class="rwa-button secondary" style="padding: 3px 8px; line-height: 17px; margin: 5px;">Cancel</button><button id="savebutton" class="rwa-button primary" style="padding: 3px 8px; line-height: 17px; margin: 5px;">Save edits</button><br><textarea id="extractedtextarea" style="width:100%;"></textarea></div>';
			reviewinterface.getElementById('extractedtextarea').value = jsn['LongTextContent'];
			reviewinterface.getElementById('extractedtextarea').style.height = (reviewinterface.getElementById('extractedtextarea').scrollHeight + 10) + 'px';

			reviewinterface.getElementById('cancelbutton').onclick = function() {
				oitpageblock.style.textAlign = '';
				oitpageblock.innerHTML = originalcontent;
			};

			reviewinterface.getElementById('savebutton').onclick = function() {
				reviewinterface.getElementById('extractedtextarea').disabled = true;
				reviewinterface.getElementById('cancelbutton').disabled = true;
				reviewinterface.getElementById('savebutton').disabled = true;
				fetch('https://' + host + '/Relativity.REST/api/Relativity.ObjectManager/v1/workspace/' + workspaceid.toString() + '/object/update', {
					'headers': {
						'accept': '*/*',
						'accept-language': 'en-US,en;q=0.9',
						'content-type': 'application/json',
						'sec-fetch-dest': 'empty',
						'sec-fetch-mode': 'cors',
						'sec-fetch-site': 'same-origin',
						'x-csrf-header': csrftoken
					},
					'referrerPolicy': 'strict-origin-when-cross-origin',
					'body': '{"request":{"Object":{"artifactId":' + documentid.toString() + '},"FieldValues":[{"Field":{"ArtifactID":' + longtextfieldid.toString() + '},"Value":' + JSON.stringify(reviewinterface.getElementById('extractedtextarea').value.replaceAll('\n', '\r\n')) + '}]},"OperationOptions":{"CallingContext":null}}',
					'method': 'POST',
					'mode': 'cors',
					'credentials': 'include'
				}).then(response => {
					location.reload();
				});
			};
		})
	})
}
```
To install this bookmarklet, create a new entry in your browser's Favorites Bar and set its URL to the following string:
```js
javascript:(function()%7Blet%20csrftoken%20%3D%20window.top.GetCsrfTokenFromPage()%3Blet%20host%20%3D%20window.top.location.host%3Blet%20urlparams%20%3D%20new%20URLSearchParams(window.top.location.search)%3Blet%20workspaceid%20%3D%20urlparams.get('AppID')%3Blet%20documentid%20%3D%20urlparams.get('DocumentID')%3Blet%20reviewinterface%20%3D%20document.getElementById('_ReviewInterface').contentWindow.document%3Blet%20fieldname%20%3D%20reviewinterface.getElementById('ctrl-ri-tab-text-viewer').children%5B0%5D.children%5B0%5D.children%5B0%5D.children%5B0%5D.innerText%3Bif%20(urlparams.get('ViewerType')%20%3D%3D%20'text')%20%7Bfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.REST%2Fapi%2FRelativity-DocumentViewer%2Fv1%2FPrivateDocumentViewerServiceManager%2FGetReviewInterfaceData'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22request%22%3A%7B%22WorkspaceId%22%3A'%20%2B%20workspaceid.toString()%20%2B%20'%2C%22ViewerPreference%22%3A7%2C%22Options%22%3A%7B%22ClientId%22%3A%22Relativity.ReviewInterface.PrepareDocuments%22%7D%2C%22DocumentIds%22%3A%5B'%20%2B%20documentid.toString()%20%2B%20'%5D%7D%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response%20%3D%3E%20%7Breturn%20response.json()%3B%7D).then(jsn%20%3D%3E%20%7Blet%20LongTextFields%20%3D%20Object.values(jsn%5B'DocumentData'%5D)%5B0%5D%5B'LongTextVisibility'%5D%3Blet%20longtextfieldid%3Bfor%20(const%20id%20in%20LongTextFields)%20%7Bif%20(LongTextFields%5Bid%5D%5B'TextFieldName'%5D%20%3D%3D%20fieldname)%20%7Blongtextfieldid%20%3D%20LongTextFields%5Bid%5D%5B'FieldArtifactId'%5D%3B%7D%7Dfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.REST%2Fapi%2FRelativity-DocumentViewer%2Fv1%2FPrivateDocumentViewerServiceManager%2FGetTextViewerContent'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22request%22%3A%7B%22WorkspaceId%22%3A'%20%2B%20workspaceid.toString()%20%2B%20'%2C%22ArtifactId%22%3A'%20%2B%20documentid.toString()%20%2B%20'%2C%22LongTextFieldArtifactId%22%3A'%20%2B%20longtextfieldid.toString()%20%2B%20'%2C%22PageSize%22%3A999999999%2C%22PageIndex%22%3A0%7D%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response%20%3D%3E%20%7Breturn%20response.json()%3B%7D).then(jsn%20%3D%3E%20%7Blet%20oitpageblock%3Bfor%20(const%20oit%20in%20reviewinterface.getElementsByClassName('ri-text-viewer')%5B0%5D.getElementsByClassName('oit-pageblock'))%20%7Bif%20(reviewinterface.getElementsByClassName('oit-pageblock')%5Boit%5D.className%20%3D%3D%20'oit-pageblock')%20%7Boitpageblock%20%3D%20reviewinterface.getElementsByClassName('ri-text-viewer')%5B0%5D.getElementsByClassName('oit-pageblock')%5Boit%5D.parentNode%3Bbreak%3B%7D%7Dlet%20originalcontent%20%3D%20oitpageblock.innerHTML%3Boitpageblock.style.textAlign%20%3D%20'center'%3Boitpageblock.innerHTML%20%3D%20'%3Cdiv%20class%3D%22oit-pageblock%22%3E%3Cbutton%20id%3D%22cancelbutton%22%20class%3D%22rwa-button%20secondary%22%20style%3D%22padding%3A%203px%208px%3B%20line-height%3A%2017px%3B%20margin%3A%205px%3B%22%3ECancel%3C%2Fbutton%3E%3Cbutton%20id%3D%22savebutton%22%20class%3D%22rwa-button%20primary%22%20style%3D%22padding%3A%203px%208px%3B%20line-height%3A%2017px%3B%20margin%3A%205px%3B%22%3ESave%20edits%3C%2Fbutton%3E%3Cbr%3E%3Ctextarea%20id%3D%22extractedtextarea%22%20style%3D%22width%3A100%25%3B%22%3E%3C%2Ftextarea%3E%3C%2Fdiv%3E'%3Breviewinterface.getElementById('extractedtextarea').value%20%3D%20jsn%5B'LongTextContent'%5D%3Breviewinterface.getElementById('extractedtextarea').style.height%20%3D%20(reviewinterface.getElementById('extractedtextarea').scrollHeight%20%2B%2010)%20%2B%20'px'%3Breviewinterface.getElementById('cancelbutton').onclick%20%3D%20function()%20%7Boitpageblock.style.textAlign%20%3D%20''%3Boitpageblock.innerHTML%20%3D%20originalcontent%3B%7D%3Breviewinterface.getElementById('savebutton').onclick%20%3D%20function()%20%7Breviewinterface.getElementById('extractedtextarea').disabled%20%3D%20true%3Breviewinterface.getElementById('cancelbutton').disabled%20%3D%20true%3Breviewinterface.getElementById('savebutton').disabled%20%3D%20true%3Bfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.REST%2Fapi%2FRelativity.ObjectManager%2Fv1%2Fworkspace%2F'%20%2B%20workspaceid.toString()%20%2B%20'%2Fobject%2Fupdate'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22request%22%3A%7B%22Object%22%3A%7B%22artifactId%22%3A'%20%2B%20documentid.toString()%20%2B%20'%7D%2C%22FieldValues%22%3A%5B%7B%22Field%22%3A%7B%22ArtifactID%22%3A'%20%2B%20longtextfieldid.toString()%20%2B%20'%7D%2C%22Value%22%3A'%20%2B%20JSON.stringify(reviewinterface.getElementById('extractedtextarea').value.replaceAll('%5Cn'%2C%20'%5Cr%5Cn'))%20%2B%20'%7D%5D%7D%2C%22OperationOptions%22%3A%7B%22CallingContext%22%3Anull%7D%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response%20%3D%3E%20%7Blocation.reload()%3B%7D)%3B%7D%3B%7D)%7D)%7D%7D)()
```

### Stop Inactivity Logout
We all know how counter-productive an unexpected logout from Relativity can be. This bookmarklet calls the `resetExpirationTimeout` function every 5 minutes, making sure your session does not expire due to inactivity. Technically, only the first line is necessary. The rest of the code changes color of several UI elements to indicate that the script has been activated.
```js
const resetExpirationTimeoutd = setInterval(window.top.relativitySessionManager.resetExpirationTimeout, 300000);
document.getElementById('nav_layout').style.backgroundColor = '#e2ebf3';
document.getElementById('nav_header').style.backgroundColor = '#e2ebf3';
document.getElementById('nav_navbar').style.backgroundColor = '#e2ebf3';
```
To install this bookmarklet, create a new entry in your browser's Favorites Bar and set its URL to the following string:
```js
javascript:(function(){const resetExpirationTimeoutd = setInterval(window.top.relativitySessionManager.resetExpirationTimeout, 300000); document.getElementById('nav_layout').style.backgroundColor = '#e2ebf3'; document.getElementById('nav_header').style.backgroundColor = '#e2ebf3'; document.getElementById('nav_navbar').style.backgroundColor = '#e2ebf3';})();
```

### Republish Processing Sets
Relativity(One) does not offer a Republish mass action on the Processing Set object. On large matters, there might be hundreds of Processing Sets at a time requiring Republish, such as after retrying document errors. This bookmarklet allows you to kick off mass republish of all currently selected Processing Sets with a nice progress bar and ability to cancel the process any point.
```js
if (document.getElementsByClassName('os-header__breadcrumbs-element-last')[0].innerText == 'Processing Sets') {
	let csrftoken = window.top.GetCsrfTokenFromPage();
	let host = window.top.location.host;
	let urlparams = new URLSearchParams(window.top.location.search);
	let workspaceid = urlparams.get('AppID');
	window.selectedkeys = [];
	window.listpage = document.getElementById('_ListPage').contentWindow.document;
	try {
		window.selectedkeys = window.listpage.getElementById('itemList').shadowRoot.querySelector('rwc-grid').selectedKeys;
	} catch {}
	let selectiondropdown = window.listpage.getElementById('selectionDropdown');
	if (selectiondropdown === null) {
		let selectedrows = window.listpage.getElementById('fil_itemListFUI').querySelectorAll('tr[aria-selected="true"]');
		for (i in selectedrows) {
			try {
				window.selectedkeys.push(selectedrows[i].querySelector('td[aria-describedby="fil_itemListFUI_ArtifactID"]').innerText);
			} catch {}
		}
		selectiondropdown = window.listpage.getElementById('massOps').getElementsByTagName('span')[0];
	}

	if (selectiondropdown.innerText.substring(0, 9) == 'Checked (' && selectiondropdown.innerText.replace(/\D/g, '') == window.selectedkeys.length) {
		let modaldialog = document.createElement('dynamic-content-modal-wgt');
		modaldialog.innerHTML = `
<style>
 #modalheader{
     font-size:18px;
     padding:15px 0 10px 20px;
     border-bottom:1px solid #e2ebf3;
     color:#3d454e 
}
 #buttoncontainer{
     position:absolute;
     bottom:20px;
     right:20px 
}
 button{
     font-size:14px;
     border:1px solid #0075e0;
     border-radius:3px;
     cursor:pointer;
     padding:6px 15px 
}
 #cancelbutton{
     background:#fff;
     color:#0075e0 
}
 #cancelbutton:focus-visible,#cancelbutton:hover{
     background:#dcf4f9 
}
 #republishbutton{
	 margin-left:10px;
     background:#0075e0;
     color:#fff 
}
 #republishbutton:focus-visible,#republishbutton:hover{
     background:#0670c1;
     border-color:#0670c1 
}
 #republishbutton:disabled, #republishbutton:disabled:hover{
     background:#f3f8fb;
     border-color:#acbfd6;
     color: #acbfd6;
     cursor: default;
}
 .rwa-progress-bar{
     color:#485769;
     display:flex;
     flex-direction:column;
     flex-wrap:nowrap;
     font-size:18px;
     line-height:20px 
}
 .rwa-progress-bar__inner-progress-container{
     background-color:#6c7584;
     bottom:0;
     border-radius:0.5rem;
     left:0;
     position:absolute;
     transition:width 0.3s linear;
     top:0 
}
 .rwa-progress-bar__main-area{
     display:flex;
     align-items:center 
}
 .rwa-progress-bar__outer-progress-container{
     background-color:#e2ebf3;
     border-radius:0.5rem;
     box-sizing:border-box;
     flex:1 0 auto;
     height:0.75rem;
     position:relative 
}
 .rwa-progress-bar__elapsed-time{
     margin-left:0.75rem 
}
 .rwa-progress-bar__percentage{
     width:3rem;
     margin-left:1rem 
}
 
</style>
<div class="modal-context first-focus-element" style="z-index: 999; background-color: rgba(108, 117, 132, 0.5);">
	<div class="modal-container" style="height: 165px;width: 600px;">
		<div id="modalheader">Republish ` + window.selectedkeys.length.toString() + ` Processing Set` + ((window.selectedkeys.length > 1) ? 's' : '') + `?</div>
		<rwc-progress-bar style="padding: 20px 20px;">
			<div class="rwa-progress-bar" id="job-progress-bar" style="display: none;">
				<div class="rwa-progress-bar__main-area">
					<div class="rwa-progress-bar__outer-progress-container">
						<div id="progressbar" class="rwa-progress-bar__inner-progress-container" style="width: 0%;"></div>
					</div>
					<span id="progresspercentage" class="rwa-progress-bar__percentage ">0%</span>
					<span id="progresstime" class="rwa-progress-bar__elapsed-time">00:00:24</span>
				</div>
			</div>
		</rwc-progress-bar>
		<div id="buttoncontainer">
			<button id="cancelbutton" onclick="document.getElementsByTagName('dynamic-content-modal-wgt')[0].remove();">Cancel</button>
			<button id="republishbutton">Republish</button>
		</div>
	</div>
</div>`;

		window.listpage.getElementById('dashboardModals').appendChild(modaldialog);
		window.listpage.getElementById('republishbutton').onclick = function() {
			let starttime = Date.now();
			window.listpage.getElementById('cancelbutton').onclick = function() {
				window.republishcancel = true;
				window.listpage.getElementsByTagName('dynamic-content-modal-wgt')[0].remove();
			};
			window.listpage.getElementById('republishbutton').disabled = true;
			window.listpage.getElementById('job-progress-bar').style.display = '';

			function timer() {
				let diff = Math.abs(Date.now() - starttime);
				let ms = diff % 1000;
				diff = (diff - ms) / 1000;
				let ss = diff % 60;
				diff = (diff - ss) / 60;
				let mm = diff % 60;
				diff = (diff - mm) / 60;
				let hh = diff % 24;
				return ('0' + hh.toString()).substr(-2) + ':' + ('0' + mm.toString()).substr(-2) + ':' + ('0' + ss.toString()).substr(-2);
			}

			async function submitrepublish() {
				for (let i = 0; i < window.selectedkeys.length; i++) {
					if (window.republishcancel) {
						window.republishcancel = false;
						break;
					}
					fetch('https://' + host + '/Relativity.Rest/API/relativity-processing/v1/workspaces/' + workspaceid.toString() + '/processing-jobs/' + window.selectedkeys[i].toString() + '/publish', {
						'headers': {
							'accept': '*/*',
							'accept-language': 'en-US,en;q=0.9',
							'content-type': 'application/json',
							'sec-fetch-dest': 'empty',
							'sec-fetch-mode': 'cors',
							'sec-fetch-site': 'same-origin',
							'x-csrf-header': csrftoken
						},
						'referrerPolicy': 'strict-origin-when-cross-origin',
						'body': '{}',
						'method': 'POST',
						'mode': 'cors',
						'credentials': 'include'
					});

					let percentagestring = Math.round(((i + 1) / window.selectedkeys.length) * 100).toString() + '%';
					window.listpage.getElementById('progressbar').style.width = percentagestring;
					window.listpage.getElementById('progresspercentage').innerText = percentagestring;
					window.listpage.getElementById('progresstime').innerText = timer();

					if ((i + 1) == window.selectedkeys.length) {
						window.listpage.getElementById('cancelbutton').innerText = 'Done';
						window.listpage.getElementById('cancelbutton').onclick = function() {
							window.listpage.getElementsByTagName('dynamic-content-modal-wgt')[0].remove();
						};
					}

					await new Promise(resolve => setTimeout(resolve, 1000));
				}
			}
			submitrepublish();
		}
	}
}
```
To install this bookmarklet, create a new entry in your browser's Favorites Bar and set its URL to the following string:
```js
javascript:(function()%7Bif%20(document.getElementsByClassName('os-header__breadcrumbs-element-last')%5B0%5D.innerText%20%3D%3D%20'Processing%20Sets')%20%7Blet%20csrftoken%20%3D%20window.top.GetCsrfTokenFromPage()%3Blet%20host%20%3D%20window.top.location.host%3Blet%20urlparams%20%3D%20new%20URLSearchParams(window.top.location.search)%3Blet%20workspaceid%20%3D%20urlparams.get('AppID')%3Bwindow.selectedkeys%20%3D%20%5B%5D%3Bwindow.listpage%20%3D%20document.getElementById('_ListPage').contentWindow.document%3Btry%20%7Bwindow.selectedkeys%20%3D%20window.listpage.getElementById('itemList').shadowRoot.querySelector('rwc-grid').selectedKeys%3B%7D%20catch%20%7B%7Dlet%20selectiondropdown%20%3D%20window.listpage.getElementById('selectionDropdown')%3Bif%20(selectiondropdown%20%3D%3D%3D%20null)%20%7Blet%20selectedrows%20%3D%20window.listpage.getElementById('fil_itemListFUI').querySelectorAll('tr%5Baria-selected%3D%22true%22%5D')%3Bfor%20(i%20in%20selectedrows)%20%7Btry%20%7Bwindow.selectedkeys.push(selectedrows%5Bi%5D.querySelector('td%5Baria-describedby%3D%22fil_itemListFUI_ArtifactID%22%5D').innerText)%3B%7D%20catch%20%7B%7D%7Dselectiondropdown%20%3D%20window.listpage.getElementById('massOps').getElementsByTagName('span')%5B0%5D%3B%7Dif%20(selectiondropdown.innerText.substring(0%2C%209)%20%3D%3D%20'Checked%20('%20%26%26%20selectiondropdown.innerText.replace(%2F%5CD%2Fg%2C%20'')%20%3D%3D%20window.selectedkeys.length)%20%7Blet%20modaldialog%20%3D%20document.createElement('dynamic-content-modal-wgt')%3Bmodaldialog.innerHTML%20%3D%20%60%3Cstyle%3E%23modalheader%7Bfont-size%3A18px%3Bpadding%3A15px%200%2010px%2020px%3Bborder-bottom%3A1px%20solid%20%23e2ebf3%3Bcolor%3A%233d454e%7D%23buttoncontainer%7Bposition%3Aabsolute%3Bbottom%3A20px%3Bright%3A20px%7Dbutton%7Bfont-size%3A14px%3Bborder%3A1px%20solid%20%230075e0%3Bborder-radius%3A3px%3Bcursor%3Apointer%3Bpadding%3A6px%2015px%7D%23cancelbutton%7Bbackground%3A%23fff%3Bcolor%3A%230075e0%7D%23cancelbutton%3Afocus-visible%2C%23cancelbutton%3Ahover%7Bbackground%3A%23dcf4f9%7D%23republishbutton%7Bmargin-left%3A10px%3Bbackground%3A%230075e0%3Bcolor%3A%23fff%7D%23republishbutton%3Afocus-visible%2C%23republishbutton%3Ahover%7Bbackground%3A%230670c1%3Bborder-color%3A%230670c1%7D%23republishbutton%3Adisabled%2C%20%23republishbutton%3Adisabled%3Ahover%7Bbackground%3A%23f3f8fb%3Bborder-color%3A%23acbfd6%3Bcolor%3A%20%23acbfd6%3Bcursor%3A%20default%3B%7D.rwa-progress-bar%7Bcolor%3A%23485769%3Bdisplay%3Aflex%3Bflex-direction%3Acolumn%3Bflex-wrap%3Anowrap%3Bfont-size%3A18px%3Bline-height%3A20px%7D.rwa-progress-bar__inner-progress-container%7Bbackground-color%3A%236c7584%3Bbottom%3A0%3Bborder-radius%3A0.5rem%3Bleft%3A0%3Bposition%3Aabsolute%3Btransition%3Awidth%200.3s%20linear%3Btop%3A0%7D.rwa-progress-bar__main-area%7Bdisplay%3Aflex%3Balign-items%3Acenter%7D.rwa-progress-bar__outer-progress-container%7Bbackground-color%3A%23e2ebf3%3Bborder-radius%3A0.5rem%3Bbox-sizing%3Aborder-box%3Bflex%3A1%200%20auto%3Bheight%3A0.75rem%3Bposition%3Arelative%7D.rwa-progress-bar__elapsed-time%7Bmargin-left%3A0.75rem%7D.rwa-progress-bar__percentage%7Bwidth%3A3rem%3Bmargin-left%3A1rem%7D%3C%2Fstyle%3E%3Cdiv%20class%3D%22modal-context%20first-focus-element%22%20style%3D%22z-index%3A%20999%3B%20background-color%3A%20rgba(108%2C%20117%2C%20132%2C%200.5)%3B%22%3E%3Cdiv%20class%3D%22modal-container%22%20style%3D%22height%3A%20165px%3Bwidth%3A%20600px%3B%22%3E%3Cdiv%20id%3D%22modalheader%22%3ERepublish%20%60%20%2B%20window.selectedkeys.length.toString()%20%2B%20%60%20Processing%20Set%60%20%2B%20((window.selectedkeys.length%20%3E%201)%20%3F%20's'%20%3A%20'')%20%2B%20%60%3F%3C%2Fdiv%3E%3Crwc-progress-bar%20style%3D%22padding%3A%2020px%2020px%3B%22%3E%3Cdiv%20class%3D%22rwa-progress-bar%22%20id%3D%22job-progress-bar%22%20style%3D%22display%3A%20none%3B%22%3E%3Cdiv%20class%3D%22rwa-progress-bar__main-area%22%3E%3Cdiv%20class%3D%22rwa-progress-bar__outer-progress-container%22%3E%3Cdiv%20id%3D%22progressbar%22%20class%3D%22rwa-progress-bar__inner-progress-container%22%20style%3D%22width%3A%200%25%3B%22%3E%3C%2Fdiv%3E%3C%2Fdiv%3E%3Cspan%20id%3D%22progresspercentage%22%20class%3D%22rwa-progress-bar__percentage%20%22%3E0%25%3C%2Fspan%3E%3Cspan%20id%3D%22progresstime%22%20class%3D%22rwa-progress-bar__elapsed-time%22%3E00%3A00%3A24%3C%2Fspan%3E%3C%2Fdiv%3E%3C%2Fdiv%3E%3C%2Frwc-progress-bar%3E%3Cdiv%20id%3D%22buttoncontainer%22%3E%3Cbutton%20id%3D%22cancelbutton%22%20onclick%3D%22document.getElementsByTagName('dynamic-content-modal-wgt')%5B0%5D.remove()%3B%22%3ECancel%3C%2Fbutton%3E%3Cbutton%20id%3D%22republishbutton%22%3ERepublish%3C%2Fbutton%3E%3C%2Fdiv%3E%3C%2Fdiv%3E%3C%2Fdiv%3E%60%3Bwindow.listpage.getElementById('dashboardModals').appendChild(modaldialog)%3Bwindow.listpage.getElementById('republishbutton').onclick%20%3D%20function()%20%7Blet%20starttime%20%3D%20Date.now()%3Bwindow.listpage.getElementById('cancelbutton').onclick%20%3D%20function()%20%7Bwindow.republishcancel%20%3D%20true%3Bwindow.listpage.getElementsByTagName('dynamic-content-modal-wgt')%5B0%5D.remove()%3B%7D%3Bwindow.listpage.getElementById('republishbutton').disabled%20%3D%20true%3Bwindow.listpage.getElementById('job-progress-bar').style.display%20%3D%20''%3Bfunction%20timer()%20%7Blet%20diff%20%3D%20Math.abs(Date.now()%20-%20starttime)%3Blet%20ms%20%3D%20diff%20%25%201000%3Bdiff%20%3D%20(diff%20-%20ms)%20%2F%201000%3Blet%20ss%20%3D%20diff%20%25%2060%3Bdiff%20%3D%20(diff%20-%20ss)%20%2F%2060%3Blet%20mm%20%3D%20diff%20%25%2060%3Bdiff%20%3D%20(diff%20-%20mm)%20%2F%2060%3Blet%20hh%20%3D%20diff%20%25%2024%3Breturn%20('0'%20%2B%20hh.toString()).substr(-2)%20%2B%20'%3A'%20%2B%20('0'%20%2B%20mm.toString()).substr(-2)%20%2B%20'%3A'%20%2B%20('0'%20%2B%20ss.toString()).substr(-2)%3B%7Dasync%20function%20submitrepublish()%20%7Bfor%20(let%20i%20%3D%200%3B%20i%20%3C%20window.selectedkeys.length%3B%20i%2B%2B)%20%7Bif%20(window.republishcancel)%20%7Bwindow.republishcancel%20%3D%20false%3Bbreak%3B%7Dfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.Rest%2FAPI%2Frelativity-processing%2Fv1%2Fworkspaces%2F'%20%2B%20workspaceid.toString()%20%2B%20'%2Fprocessing-jobs%2F'%20%2B%20window.selectedkeys%5Bi%5D.toString()%20%2B%20'%2Fpublish'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D)%3Blet%20percentagestring%20%3D%20Math.round(((i%20%2B%201)%20%2F%20window.selectedkeys.length)%20*%20100).toString()%20%2B%20'%25'%3Bwindow.listpage.getElementById('progressbar').style.width%20%3D%20percentagestring%3Bwindow.listpage.getElementById('progresspercentage').innerText%20%3D%20percentagestring%3Bwindow.listpage.getElementById('progresstime').innerText%20%3D%20timer()%3Bif%20((i%20%2B%201)%20%3D%3D%20window.selectedkeys.length)%20%7Bwindow.listpage.getElementById('cancelbutton').innerText%20%3D%20'Done'%3Bwindow.listpage.getElementById('cancelbutton').onclick%20%3D%20function()%20%7Bwindow.listpage.getElementsByTagName('dynamic-content-modal-wgt')%5B0%5D.remove()%3B%7D%3B%7Dawait%20new%20Promise(resolve%20%3D%3E%20setTimeout(resolve%2C%201000))%3B%7D%7Dsubmitrepublish()%3B%7D%7D%7D%7D)()
```

### Select All Searches
Before you can [Export Saved Searches](#export-saved-searches), you need to manually select which searches to export, which can mean tens or hundreds of clicks and a potential for mistake. This bookmarklet selects all currently displayed searches in one go.
```js
let listpage = document.getElementById('_ListPage').contentWindow.document;
let searches = [];

try {
	if (listpage.querySelector('button[title="Show Checkboxes"]') !== null) {
		listpage.querySelector('button[title="Show Checkboxes"]').click();
	}
} catch {}

try {
	if (listpage.querySelector('div[name="savedSearch"]').querySelector('button.btn-show-checkbox').className == 'btn-show-checkbox') {
		listpage.querySelector('div[name="savedSearch"]').querySelector('button.btn-show-checkbox').click();
	}
} catch {}

setTimeout(function() {
	if (listpage.querySelector('rwc-tab[title="Saved Searches"][active]') !== null) {
		searches = listpage.querySelector('rwc-tree').querySelectorAll('rwc-tree-node[icon-name="search"]');
			for (let i = 0; i < searches.length; i++) {
				try {
					searches[i].shadowRoot.querySelector('rwc-boolean-checkbox-input[value="false"]').shadowRoot.querySelector('span.rwa-checkbox').click();
				} catch {}
			}
	} else if (listpage.querySelector('a.browser.browser-active[title="Saved Searches"]') !== null) {
		searches = listpage.querySelector('div[name="savedSearch"]').querySelectorAll('li.jstree-leaf');
		for (let i = 0; i < searches.length; i++) {
			try {
				if (!searches[i].querySelector('a').className.includes('jstree-checked')) {
					searches[i].querySelector('a').querySelector('i.jstree-icon.jstree-checkbox').click();
				}
			} catch {}
		}
	}
}, 100);
```
To install this bookmarklet, create a new entry in your browser's Favorites Bar and set its URL to the following string:
```js
javascript:(function()%7Blet%20listpage%20%3D%20document.getElementById('_ListPage').contentWindow.document%3Blet%20searches%20%3D%20%5B%5D%3Btry%20%7Bif%20(listpage.querySelector('button%5Btitle%3D%22Show%20Checkboxes%22%5D')%20!%3D%3D%20null)%20%7Blistpage.querySelector('button%5Btitle%3D%22Show%20Checkboxes%22%5D').click()%3B%7D%7D%20catch%20%7B%7Dtry%20%7Bif%20(listpage.querySelector('div%5Bname%3D%22savedSearch%22%5D').querySelector('button.btn-show-checkbox').className%20%3D%3D%20'btn-show-checkbox')%20%7Blistpage.querySelector('div%5Bname%3D%22savedSearch%22%5D').querySelector('button.btn-show-checkbox').click()%3B%7D%7D%20catch%20%7B%7DsetTimeout(function()%20%7Bif%20(listpage.querySelector('rwc-tab%5Btitle%3D%22Saved%20Searches%22%5D%5Bactive%5D')%20!%3D%3D%20null)%20%7Bsearches%20%3D%20listpage.querySelector('rwc-tree').querySelectorAll('rwc-tree-node%5Bicon-name%3D%22search%22%5D')%3Bfor%20(let%20i%20%3D%200%3B%20i%20%3C%20searches.length%3B%20i%2B%2B)%20%7Btry%20%7Bsearches%5Bi%5D.shadowRoot.querySelector('rwc-boolean-checkbox-input%5Bvalue%3D%22false%22%5D').shadowRoot.querySelector('span.rwa-checkbox').click()%3B%7D%20catch%20%7B%7D%7D%7D%20else%20if%20(listpage.querySelector('a.browser.browser-active%5Btitle%3D%22Saved%20Searches%22%5D')%20!%3D%3D%20null)%20%7Bsearches%20%3D%20listpage.querySelector('div%5Bname%3D%22savedSearch%22%5D').querySelectorAll('li.jstree-leaf')%3Bfor%20(let%20i%20%3D%200%3B%20i%20%3C%20searches.length%3B%20i%2B%2B)%20%7Btry%20%7Bif%20(!searches%5Bi%5D.querySelector('a').className.includes('jstree-checked'))%20%7Bsearches%5Bi%5D.querySelector('a').querySelector('i.jstree-icon.jstree-checkbox').click()%3B%7D%7D%20catch%20%7B%7D%7D%7D%7D%2C%20100)%7D)()
```

### Export Saved Searches
This bookmarklet produces a CSV file with search conditions of all saved searches currently selected in the search browser sidebar. This can be very useful for QC of large amount of searches that should have mostly consistent conditions.
```js
window.listpage = document.getElementById('_ListPage').contentWindow.document;
if (window.listpage.querySelector('rwc-tab[title="Saved Searches"][active]') !== null || window.listpage.querySelector('a.browser.browser-active[title="Saved Searches"]') !== null) {
	let csrftoken = window.top.GetCsrfTokenFromPage();
	let host = window.top.location.host;
	let urlparams = new URLSearchParams(window.top.location.search);
	let workspaceid = urlparams.get('AppID');
	if (window.listpage.querySelector('rwc-tree') !== null) {
		window.newui = true;
	} else {
		window.newui = false;
	}
	window.searchjsons = [];
	window.searches = [];
	if (window.newui) {
		let nodes = window.listpage.querySelector('rwc-tree').querySelectorAll('rwc-tree-node[icon-name="search"][check-value="true"]');
		for (let i = 0; i < nodes.length; i++) {
			try {
				window.searches.push(nodes[i].value);
			} catch {}
		}
	} else {
		let nodes = window.listpage.querySelector('div[name="savedSearch"]').querySelectorAll('li.jstree-leaf');
		for (let i = 0; i < nodes.length; i++) {
			try {
				window.searches.push(nodes[i].querySelector('a.jstree-checked').attributes.id.value.replace('_anchor', ''));
			} catch {}
		}
	}
	if (window.searches.length == 0) {
		alert('No Saved Searches have been selected for export.');
		if (window.newui) {
			window.listpage.querySelector('button[title="Show Checkboxes"]').click();
		} else {
			window.listpage.querySelector('div[name="savedSearch"]').querySelector('button.btn-show-checkbox').click();
		}
	} else {
		window.exportcancel = false;
		if (window.newui) {
			window.listpage = document.getElementById('_ListPage').contentWindow.document;

			let modaldialog = document.createElement('ux-dialog-container');
			modaldialog.innerHTML = `
<div>
	<div>
		<rwc-modal-layout class="small" id="loadingModal" theme="" vertical-margin="0" defined="">
			<span slot="header" class="au-target">
				<span>Export ` + window.searches.length.toString() + ` Saved Search` + ((window.searches.length > 1) ? 'es' : '') + `?</span>
			</span>
			<span slot="content" class="container au-target" style="display: none;">
				<rwc-progress-bar id="progressbar" style="width: 65%; display: inline-block; margin-top: 4px;"></rwc-progress-bar>
				<span class="rwa-progress-bar" id="job-progress-bar" style="display: inline-block; float: right;">
						<span id="progresspercentage" class="biggerText" style="margin-right: 10px;">0%</span>
						<span id="progresstime" class="biggerText">00:00:00</span>
				</span>
			</span>
			<span slot="actions" class="au-target">
				<button id="cancelbutton" class="rwa-button secondary au-target" onclick="document.getElementsByTagName('ux-dialog-overlay')[0].remove(); document.getElementsByTagName('ux-dialog-container')[0].remove();" style="margin-right: 10px;">Cancel</button>
				<button id="exportbutton" class="rwa-button primary au-target">Export</button>
			</span>
		</rwc-modal-layout>
	</div>
</div>`;
			modaldialog.className = 'active';
			modaldialog.style = 'z-index: 999;';
			window.listpage.getElementsByTagName('body')[0].prepend(modaldialog);

			let modaloverlay = document.createElement('ux-dialog-overlay');
			modaloverlay.className = 'active';
			modaloverlay.style = 'z-index: 999;';
			window.listpage.getElementsByTagName('body')[0].prepend(modaloverlay);
		} else {
			let modaldialog = document.createElement('dynamic-content-modal-wgt');
			modaldialog.innerHTML = `
<style>
 #modalheader{
     font-size:18px;
     padding:15px 0 10px 20px;
     border-bottom:1px solid #e2ebf3;
     color:#3d454e 
}
 #buttoncontainer{
     position:absolute;
     bottom:20px;
     right:20px 
}
 button{
     font-size:14px;
     border:1px solid #0075e0;
     border-radius:3px;
     cursor:pointer;
     padding:6px 15px 
}
 #cancelbutton{
     background:#fff;
     color:#0075e0 
}
 #cancelbutton:focus-visible,#cancelbutton:hover{
     background:#dcf4f9 
}
 #exportbutton{
	 margin-left:10px;
     background:#0075e0;
     color:#fff 
}
 #exportbutton:focus-visible,#exportbutton:hover{
     background:#0670c1;
     border-color:#0670c1 
}
 #exportbutton:disabled, #exportbutton:disabled:hover{
     background:#f3f8fb;
     border-color:#acbfd6;
     color: #acbfd6;
     cursor: default;
}
 .rwa-progress-bar{
     color:#485769;
     display:flex;
     flex-direction:column;
     flex-wrap:nowrap;
     font-size:18px;
     line-height:20px 
}
 .rwa-progress-bar__inner-progress-container{
     background-color:#6c7584;
     bottom:0;
     border-radius:0.5rem;
     left:0;
     position:absolute;
     transition:width 0.3s linear;
     top:0 
}
 .rwa-progress-bar__main-area{
     display:flex;
     align-items:center 
}
 .rwa-progress-bar__outer-progress-container{
     background-color:#e2ebf3;
     border-radius:0.5rem;
     box-sizing:border-box;
     flex:1 0 auto;
     height:0.75rem;
     position:relative 
}
 .rwa-progress-bar__elapsed-time{
     margin-left:0.75rem 
}
 .rwa-progress-bar__percentage{
     width:3rem;
     margin-left:1rem 
}
 
</style>
<div class="modal-context first-focus-element" style="z-index: 999; background-color: rgba(108, 117, 132, 0.5);">
	<div class="modal-container" style="height: 165px; width: 600px;">
		<div id="modalheader">Export ` + window.searches.length.toString() + ` Saved Search` + ((window.searches.length > 1) ? 'es' : '') + `?</div>
		<rwc-progress-bar style="padding: 20px 20px;">
			<div class="rwa-progress-bar" id="job-progress-bar" style="display: none;">
				<div class="rwa-progress-bar__main-area">
					<div class="rwa-progress-bar__outer-progress-container">
						<div id="progressbar" class="rwa-progress-bar__inner-progress-container" style="width: 0%;"></div>
					</div>
					<span id="progresspercentage" class="rwa-progress-bar__percentage ">0%</span>
					<span id="progresstime" class="rwa-progress-bar__elapsed-time">00:00:00</span>
				</div>
			</div>
		</rwc-progress-bar>
		<div id="buttoncontainer">
			<button id="cancelbutton" onclick="document.getElementsByTagName('dynamic-content-modal-wgt')[0].remove();">Cancel</button>
			<button id="exportbutton">Export</button>
		</div>
	</div>
</div>`;
			window.listpage.getElementById('layoutDashboard').appendChild(modaldialog);
		}
		window.listpage.getElementById('exportbutton').onclick = function() {
			let starttime = Date.now();
			window.exportcancel = false;

			if (window.newui) {
				window.listpage.getElementById('cancelbutton').onclick = function() {
					window.exportcancel = true;
					window.listpage.getElementsByTagName('ux-dialog-overlay')[0].remove();
					window.listpage.getElementsByTagName('ux-dialog-container')[0].remove();
				};
				window.listpage.getElementById('exportbutton').disabled = true;
				window.listpage.querySelector('span[slot="content"]').style.display = '';
			} else {
				window.listpage.getElementById('cancelbutton').onclick = function() {
					window.exportcancel = true;
					window.listpage.getElementsByTagName('dynamic-content-modal-wgt')[0].remove();
				};
				window.listpage.getElementById('exportbutton').disabled = true;
				window.listpage.getElementById('progressbar').style.display = '';
				window.listpage.getElementById('job-progress-bar').style.display = '';
			}

			function timer() {
				let diff = Math.abs(Date.now() - starttime);
				let ms = diff % 1000;
				diff = (diff - ms) / 1000;
				let ss = diff % 60;
				diff = (diff - ss) / 60;
				let mm = diff % 60;
				diff = (diff - mm) / 60;
				let hh = diff % 24;
				return ('0' + hh.toString()).substr(-2) + ':' + ('0' + mm.toString()).substr(-2) + ':' + ('0' + ss.toString()).substr(-2);
			}

			async function startexport() {
				for (let i = 0; i < window.searches.length; i++) {
					if (window.exportcancel) {
						break;
					}
					fetch('https://' + host + '/Relativity.Rest/API/Relativity.Services.Search.ISearchModule/Keyword%20Search%20Manager/ReadSingleAsync', {
						'headers': {
							'accept': '*/*',
							'accept-language': 'en-US,en;q=0.9',
							'content-type': 'application/json',
							'sec-fetch-dest': 'empty',
							'sec-fetch-mode': 'cors',
							'sec-fetch-site': 'same-origin',
							'x-csrf-header': csrftoken
						},
						'referrerPolicy': 'strict-origin-when-cross-origin',
						'body': '{"workspaceArtifactID":' + workspaceid.toString() + ',"searchArtifactID":' + window.searches[i].toString() + '}',
						'method': 'POST',
						'mode': 'cors',
						'credentials': 'include'
					}).then(response => {
						return response.json();
					}).then(jsn => {
						if (JSON.stringify(jsn) == '{"ErrorType":"Relativity.Services.Exceptions.ServiceException","Identifier":"","Message":"Artifact matching that Artifact ID not found."}') {
							fetch('https://' + host + '/Relativity.Rest/API/Relativity.Services.Search.ISearchModule/Analytics%20Search%20Manager/ReadSingleAsync', {
								'headers': {
									'accept': '*/*',
									'accept-language': 'en-US,en;q=0.9',
									'content-type': 'application/json',
									'sec-fetch-dest': 'empty',
									'sec-fetch-mode': 'cors',
									'sec-fetch-site': 'same-origin',
									'x-csrf-header': csrftoken
								},
								'referrerPolicy': 'strict-origin-when-cross-origin',
								'body': '{"workspaceArtifactID":' + workspaceid.toString() + ',"searchArtifactID":' + window.searches[i].toString() + '}',
								'method': 'POST',
								'mode': 'cors',
								'credentials': 'include'
							}).then(response => {
								return response.json();
							}).then(jsn => {
								if (JSON.stringify(jsn) == '{"ErrorType":"Relativity.Services.Exceptions.ServiceException","Identifier":"","Message":"Artifact matching that Artifact ID not found."}') {
									fetch('https://' + host + '/Relativity.Rest/API/Relativity.Services.Search.ISearchModule/dtSearch%20Manager/ReadSingleAsync', {
										'headers': {
											'accept': '*/*',
											'accept-language': 'en-US,en;q=0.9',
											'content-type': 'application/json',
											'sec-fetch-dest': 'empty',
											'sec-fetch-mode': 'cors',
											'sec-fetch-site': 'same-origin',
											'x-csrf-header': csrftoken
										},
										'referrerPolicy': 'strict-origin-when-cross-origin',
										'body': '{"workspaceArtifactID":' + workspaceid.toString() + ',"searchArtifactID":' + window.searches[i].toString() + '}',
										'method': 'POST',
										'mode': 'cors',
										'credentials': 'include'
									}).then(response => {
										return response.json();
									}).then(jsn => {
										window.searchjsons.push(jsn);
										if (window.searchjsons.length == window.searches.length) {
											downloadCSV();
										}
									});
								} else {
									window.searchjsons.push(jsn);
									if (window.searchjsons.length == window.searches.length) {
										downloadCSV();
									}
								}
							});
						} else {
							window.searchjsons.push(jsn);
							if (window.searchjsons.length == window.searches.length) {
								downloadCSV();
							}
						}
					});
					await new Promise(resolve => setTimeout(resolve, 100));
				}
			}

			window.updatetimer = setInterval(updateProgress, 1000);
			startexport();

			function updateProgress() {
				if (window.searchjsons.length == window.searches.length) {
					clearInterval(window.updatetimer);
					window.listpage.getElementById('cancelbutton').innerText = 'Done';
				} else if (window.exportcancel) {
					clearInterval(window.updatetimer);
				}
				try {
					let percentage = Math.floor((window.searchjsons.length / window.searches.length) * 100);
					if (window.newui) {
						window.listpage.getElementById('progressbar').value = percentage;
					} else {
						window.listpage.getElementById('progressbar').style.width = percentage.toString() + '%';
					}
					window.listpage.getElementById('progresspercentage').innerText = percentage.toString() + '%';
					window.listpage.getElementById('progresstime').innerText = timer();
				} catch {}
			}

			function isScalar(value) {
				return (/string|number|boolean/).test(typeof value);
			}

			function flattenObject(obj, prefix = '') {
				let result = {};
				for (const [key, value] of Object.entries(obj)) {
					const newKey = prefix ? `${prefix}.${key}` : key;
					if (Array.isArray(value)) {
						if (value.every(isScalar)) {
							result[newKey] = value.join('; ');
						} else {
							for (let i = 0; i < value.length; i++) {
								const tempResult = flattenObject(value[i], `${newKey}.${i}`);
								result = {
									...result,
									...tempResult
								};
							}
						}
					} else if (typeof value === 'object' && value !== null) {
						const tempResult = flattenObject(value, newKey);
						result = {
							...result,
							...tempResult
						};
					} else {
						result[newKey] = value;
					}
				}
				return result;
			}

			function getAllKeys(data) {
				let keysSet = new Set();
				data.forEach(item => {
					const flattened = flattenObject(item);
					Object.keys(flattened).forEach(key => keysSet.add(key));
				});
				return Array.from(keysSet);
			}

			function generateCSV(data) {
				const keys = getAllKeys(data);
				const flattenedData = data.map(item => flattenObject(item));

				const csvRows = flattenedData.map(obj => {
					return keys.map(key => `"${String(obj[key] ?? '').replace(/"/g, '""')}"`).join(',');
				});

				return [keys.join(','), ...csvRows].join('\n');
			}

			function downloadCSV() {
				let blob = new Blob(['\ufeff', generateCSV(searchjsons)], {
					type: 'text/csv'
				});
				let objectUrl = URL.createObjectURL(blob);
				let dlink = document.createElement('a');
				dlink.setAttribute('href', objectUrl);
				dlink.setAttribute('download', 'Saved searches.csv');
				dlink.click();
			}

		}
	}
}
```
To install this bookmarklet, create a new entry in your browser's Favorites Bar and set its URL to the following string:
```js
javascript:(function()%7Bwindow.listpage%20%3D%20document.getElementById('_ListPage').contentWindow.document%3Bif%20(window.listpage.querySelector('rwc-tab%5Btitle%3D%22Saved%20Searches%22%5D%5Bactive%5D')%20!%3D%3D%20null%20%7C%7C%20window.listpage.querySelector('a.browser.browser-active%5Btitle%3D%22Saved%20Searches%22%5D')%20!%3D%3D%20null)%20%7Blet%20csrftoken%20%3D%20window.top.GetCsrfTokenFromPage()%3Blet%20host%20%3D%20window.top.location.host%3Blet%20urlparams%20%3D%20new%20URLSearchParams(window.top.location.search)%3Blet%20workspaceid%20%3D%20urlparams.get('AppID')%3Bif%20(window.listpage.querySelector('rwc-tree')%20!%3D%3D%20null)%20%7Bwindow.newui%20%3D%20true%3B%7D%20else%20%7Bwindow.newui%20%3D%20false%3B%7Dwindow.searchjsons%20%3D%20%5B%5D%3Bwindow.searches%20%3D%20%5B%5D%3Bif%20(window.newui)%20%7Blet%20nodes%20%3D%20window.listpage.querySelector('rwc-tree').querySelectorAll('rwc-tree-node%5Bicon-name%3D%22search%22%5D%5Bcheck-value%3D%22true%22%5D')%3Bfor%20(let%20i%20%3D%200%3B%20i%20%3C%20nodes.length%3B%20i%2B%2B)%20%7Btry%20%7Bwindow.searches.push(nodes%5Bi%5D.value)%3B%7D%20catch%20%7B%7D%7D%7D%20else%20%7Blet%20nodes%20%3D%20window.listpage.querySelector('div%5Bname%3D%22savedSearch%22%5D').querySelectorAll('li.jstree-leaf')%3Bfor%20(let%20i%20%3D%200%3B%20i%20%3C%20nodes.length%3B%20i%2B%2B)%20%7Btry%20%7Bwindow.searches.push(nodes%5Bi%5D.querySelector('a.jstree-checked').attributes.id.value.replace('_anchor'%2C%20''))%3B%7D%20catch%20%7B%7D%7D%7Dif%20(window.searches.length%20%3D%3D%200)%20%7Balert('No%20Saved%20Searches%20have%20been%20selected%20for%20export.')%3Bif%20(window.newui)%20%7Bwindow.listpage.querySelector('button%5Btitle%3D%22Show%20Checkboxes%22%5D').click()%3B%7D%20else%20%7Bwindow.listpage.querySelector('div%5Bname%3D%22savedSearch%22%5D').querySelector('button.btn-show-checkbox').click()%3B%7D%7D%20else%20%7Bwindow.exportcancel%20%3D%20false%3Bif%20(window.newui)%20%7Bwindow.listpage%20%3D%20document.getElementById('_ListPage').contentWindow.document%3Blet%20modaldialog%20%3D%20document.createElement('ux-dialog-container')%3Bmodaldialog.innerHTML%20%3D%20%60%3Cdiv%3E%3Cdiv%3E%3Crwc-modal-layout%20class%3D%22small%22%20id%3D%22loadingModal%22%20theme%3D%22%22%20vertical-margin%3D%220%22%20defined%3D%22%22%3E%3Cspan%20slot%3D%22header%22%20class%3D%22au-target%22%3E%3Cspan%3EExport%20%60%20%2B%20window.searches.length.toString()%20%2B%20%60%20Saved%20Search%60%20%2B%20((window.searches.length%20%3E%201)%20%3F%20'es'%20%3A%20'')%20%2B%20%60%3F%3C%2Fspan%3E%3C%2Fspan%3E%3Cspan%20slot%3D%22content%22%20class%3D%22container%20au-target%22%20style%3D%22display%3A%20none%3B%22%3E%3Crwc-progress-bar%20id%3D%22progressbar%22%20style%3D%22width%3A%2065%25%3B%20display%3A%20inline-block%3B%20margin-top%3A%204px%3B%22%3E%3C%2Frwc-progress-bar%3E%3Cspan%20class%3D%22rwa-progress-bar%22%20id%3D%22job-progress-bar%22%20style%3D%22display%3A%20inline-block%3B%20float%3A%20right%3B%22%3E%3Cspan%20id%3D%22progresspercentage%22%20class%3D%22biggerText%22%20style%3D%22margin-right%3A%2010px%3B%22%3E0%25%3C%2Fspan%3E%3Cspan%20id%3D%22progresstime%22%20class%3D%22biggerText%22%3E00%3A00%3A00%3C%2Fspan%3E%3C%2Fspan%3E%3C%2Fspan%3E%3Cspan%20slot%3D%22actions%22%20class%3D%22au-target%22%3E%3Cbutton%20id%3D%22cancelbutton%22%20class%3D%22rwa-button%20secondary%20au-target%22%20onclick%3D%22document.getElementsByTagName('ux-dialog-overlay')%5B0%5D.remove()%3B%20document.getElementsByTagName('ux-dialog-container')%5B0%5D.remove()%3B%22%20style%3D%22margin-right%3A%2010px%3B%22%3ECancel%3C%2Fbutton%3E%3Cbutton%20id%3D%22exportbutton%22%20class%3D%22rwa-button%20primary%20au-target%22%3EExport%3C%2Fbutton%3E%3C%2Fspan%3E%3C%2Frwc-modal-layout%3E%3C%2Fdiv%3E%3C%2Fdiv%3E%60%3Bmodaldialog.className%20%3D%20'active'%3Bmodaldialog.style%20%3D%20'z-index%3A%20999%3B'%3Bwindow.listpage.getElementsByTagName('body')%5B0%5D.prepend(modaldialog)%3Blet%20modaloverlay%20%3D%20document.createElement('ux-dialog-overlay')%3Bmodaloverlay.className%20%3D%20'active'%3Bmodaloverlay.style%20%3D%20'z-index%3A%20999%3B'%3Bwindow.listpage.getElementsByTagName('body')%5B0%5D.prepend(modaloverlay)%3B%7D%20else%20%7Blet%20modaldialog%20%3D%20document.createElement('dynamic-content-modal-wgt')%3Bmodaldialog.innerHTML%20%3D%20%60%3Cstyle%3E%23modalheader%7Bfont-size%3A18px%3Bpadding%3A15px%200%2010px%2020px%3Bborder-bottom%3A1px%20solid%20%23e2ebf3%3Bcolor%3A%233d454e%7D%23buttoncontainer%7Bposition%3Aabsolute%3Bbottom%3A20px%3Bright%3A20px%7Dbutton%7Bfont-size%3A14px%3Bborder%3A1px%20solid%20%230075e0%3Bborder-radius%3A3px%3Bcursor%3Apointer%3Bpadding%3A6px%2015px%7D%23cancelbutton%7Bbackground%3A%23fff%3Bcolor%3A%230075e0%7D%23cancelbutton%3Afocus-visible%2C%23cancelbutton%3Ahover%7Bbackground%3A%23dcf4f9%7D%23exportbutton%7Bmargin-left%3A10px%3Bbackground%3A%230075e0%3Bcolor%3A%23fff%7D%23exportbutton%3Afocus-visible%2C%23exportbutton%3Ahover%7Bbackground%3A%230670c1%3Bborder-color%3A%230670c1%7D%23exportbutton%3Adisabled%2C%20%23exportbutton%3Adisabled%3Ahover%7Bbackground%3A%23f3f8fb%3Bborder-color%3A%23acbfd6%3Bcolor%3A%20%23acbfd6%3Bcursor%3A%20default%3B%7D.rwa-progress-bar%7Bcolor%3A%23485769%3Bdisplay%3Aflex%3Bflex-direction%3Acolumn%3Bflex-wrap%3Anowrap%3Bfont-size%3A18px%3Bline-height%3A20px%7D.rwa-progress-bar__inner-progress-container%7Bbackground-color%3A%236c7584%3Bbottom%3A0%3Bborder-radius%3A0.5rem%3Bleft%3A0%3Bposition%3Aabsolute%3Btransition%3Awidth%200.3s%20linear%3Btop%3A0%7D.rwa-progress-bar__main-area%7Bdisplay%3Aflex%3Balign-items%3Acenter%7D.rwa-progress-bar__outer-progress-container%7Bbackground-color%3A%23e2ebf3%3Bborder-radius%3A0.5rem%3Bbox-sizing%3Aborder-box%3Bflex%3A1%200%20auto%3Bheight%3A0.75rem%3Bposition%3Arelative%7D.rwa-progress-bar__elapsed-time%7Bmargin-left%3A0.75rem%7D.rwa-progress-bar__percentage%7Bwidth%3A3rem%3Bmargin-left%3A1rem%7D%3C%2Fstyle%3E%3Cdiv%20class%3D%22modal-context%20first-focus-element%22%20style%3D%22z-index%3A%20999%3B%20background-color%3A%20rgba(108%2C%20117%2C%20132%2C%200.5)%3B%22%3E%3Cdiv%20class%3D%22modal-container%22%20style%3D%22height%3A%20165px%3B%20width%3A%20600px%3B%22%3E%3Cdiv%20id%3D%22modalheader%22%3EExport%20%60%20%2B%20window.searches.length.toString()%20%2B%20%60%20Saved%20Search%60%20%2B%20((window.searches.length%20%3E%201)%20%3F%20'es'%20%3A%20'')%20%2B%20%60%3F%3C%2Fdiv%3E%3Crwc-progress-bar%20style%3D%22padding%3A%2020px%2020px%3B%22%3E%3Cdiv%20class%3D%22rwa-progress-bar%22%20id%3D%22job-progress-bar%22%20style%3D%22display%3A%20none%3B%22%3E%3Cdiv%20class%3D%22rwa-progress-bar__main-area%22%3E%3Cdiv%20class%3D%22rwa-progress-bar__outer-progress-container%22%3E%3Cdiv%20id%3D%22progressbar%22%20class%3D%22rwa-progress-bar__inner-progress-container%22%20style%3D%22width%3A%200%25%3B%22%3E%3C%2Fdiv%3E%3C%2Fdiv%3E%3Cspan%20id%3D%22progresspercentage%22%20class%3D%22rwa-progress-bar__percentage%20%22%3E0%25%3C%2Fspan%3E%3Cspan%20id%3D%22progresstime%22%20class%3D%22rwa-progress-bar__elapsed-time%22%3E00%3A00%3A00%3C%2Fspan%3E%3C%2Fdiv%3E%3C%2Fdiv%3E%3C%2Frwc-progress-bar%3E%3Cdiv%20id%3D%22buttoncontainer%22%3E%3Cbutton%20id%3D%22cancelbutton%22%20onclick%3D%22document.getElementsByTagName('dynamic-content-modal-wgt')%5B0%5D.remove()%3B%22%3ECancel%3C%2Fbutton%3E%3Cbutton%20id%3D%22exportbutton%22%3EExport%3C%2Fbutton%3E%3C%2Fdiv%3E%3C%2Fdiv%3E%3C%2Fdiv%3E%60%3Bwindow.listpage.getElementById('layoutDashboard').appendChild(modaldialog)%3B%7Dwindow.listpage.getElementById('exportbutton').onclick%20%3D%20function()%20%7Blet%20starttime%20%3D%20Date.now()%3Bwindow.exportcancel%20%3D%20false%3Bif%20(window.newui)%20%7Bwindow.listpage.getElementById('cancelbutton').onclick%20%3D%20function()%20%7Bwindow.exportcancel%20%3D%20true%3Bwindow.listpage.getElementsByTagName('ux-dialog-overlay')%5B0%5D.remove()%3Bwindow.listpage.getElementsByTagName('ux-dialog-container')%5B0%5D.remove()%3B%7D%3Bwindow.listpage.getElementById('exportbutton').disabled%20%3D%20true%3Bwindow.listpage.querySelector('span%5Bslot%3D%22content%22%5D').style.display%20%3D%20''%3B%7D%20else%20%7Bwindow.listpage.getElementById('cancelbutton').onclick%20%3D%20function()%20%7Bwindow.exportcancel%20%3D%20true%3Bwindow.listpage.getElementsByTagName('dynamic-content-modal-wgt')%5B0%5D.remove()%3B%7D%3Bwindow.listpage.getElementById('exportbutton').disabled%20%3D%20true%3Bwindow.listpage.getElementById('progressbar').style.display%20%3D%20''%3Bwindow.listpage.getElementById('job-progress-bar').style.display%20%3D%20''%3B%7Dfunction%20timer()%20%7Blet%20diff%20%3D%20Math.abs(Date.now()%20-%20starttime)%3Blet%20ms%20%3D%20diff%20%25%201000%3Bdiff%20%3D%20(diff%20-%20ms)%20%2F%201000%3Blet%20ss%20%3D%20diff%20%25%2060%3Bdiff%20%3D%20(diff%20-%20ss)%20%2F%2060%3Blet%20mm%20%3D%20diff%20%25%2060%3Bdiff%20%3D%20(diff%20-%20mm)%20%2F%2060%3Blet%20hh%20%3D%20diff%20%25%2024%3Breturn%20('0'%20%2B%20hh.toString()).substr(-2)%20%2B%20'%3A'%20%2B%20('0'%20%2B%20mm.toString()).substr(-2)%20%2B%20'%3A'%20%2B%20('0'%20%2B%20ss.toString()).substr(-2)%3B%7Dasync%20function%20startexport()%20%7Bfor%20(let%20i%20%3D%200%3B%20i%20%3C%20window.searches.length%3B%20i%2B%2B)%20%7Bif%20(window.exportcancel)%20%7Bbreak%3B%7Dfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.Rest%2FAPI%2FRelativity.Services.Search.ISearchModule%2FKeyword%2520Search%2520Manager%2FReadSingleAsync'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22workspaceArtifactID%22%3A'%20%2B%20workspaceid.toString()%20%2B%20'%2C%22searchArtifactID%22%3A'%20%2B%20window.searches%5Bi%5D.toString()%20%2B%20'%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response%20%3D%3E%20%7Breturn%20response.json()%3B%7D).then(jsn%20%3D%3E%20%7Bif%20(JSON.stringify(jsn)%20%3D%3D%20'%7B%22ErrorType%22%3A%22Relativity.Services.Exceptions.ServiceException%22%2C%22Identifier%22%3A%22%22%2C%22Message%22%3A%22Artifact%20matching%20that%20Artifact%20ID%20not%20found.%22%7D')%20%7Bfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.Rest%2FAPI%2FRelativity.Services.Search.ISearchModule%2FAnalytics%2520Search%2520Manager%2FReadSingleAsync'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22workspaceArtifactID%22%3A'%20%2B%20workspaceid.toString()%20%2B%20'%2C%22searchArtifactID%22%3A'%20%2B%20window.searches%5Bi%5D.toString()%20%2B%20'%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response%20%3D%3E%20%7Breturn%20response.json()%3B%7D).then(jsn%20%3D%3E%20%7Bif%20(JSON.stringify(jsn)%20%3D%3D%20'%7B%22ErrorType%22%3A%22Relativity.Services.Exceptions.ServiceException%22%2C%22Identifier%22%3A%22%22%2C%22Message%22%3A%22Artifact%20matching%20that%20Artifact%20ID%20not%20found.%22%7D')%20%7Bfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.Rest%2FAPI%2FRelativity.Services.Search.ISearchModule%2FdtSearch%2520Manager%2FReadSingleAsync'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22workspaceArtifactID%22%3A'%20%2B%20workspaceid.toString()%20%2B%20'%2C%22searchArtifactID%22%3A'%20%2B%20window.searches%5Bi%5D.toString()%20%2B%20'%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response%20%3D%3E%20%7Breturn%20response.json()%3B%7D).then(jsn%20%3D%3E%20%7Bwindow.searchjsons.push(jsn)%3Bif%20(window.searchjsons.length%20%3D%3D%20window.searches.length)%20%7BdownloadCSV()%3B%7D%7D)%3B%7D%20else%20%7Bwindow.searchjsons.push(jsn)%3Bif%20(window.searchjsons.length%20%3D%3D%20window.searches.length)%20%7BdownloadCSV()%3B%7D%7D%7D)%3B%7D%20else%20%7Bwindow.searchjsons.push(jsn)%3Bif%20(window.searchjsons.length%20%3D%3D%20window.searches.length)%20%7BdownloadCSV()%3B%7D%7D%7D)%3Bawait%20new%20Promise(resolve%20%3D%3E%20setTimeout(resolve%2C%20100))%3B%7D%7Dwindow.updatetimer%20%3D%20setInterval(updateProgress%2C%201000)%3Bstartexport()%3Bfunction%20updateProgress()%20%7Bif%20(window.searchjsons.length%20%3D%3D%20window.searches.length)%20%7BclearInterval(window.updatetimer)%3Bwindow.listpage.getElementById('cancelbutton').innerText%20%3D%20'Done'%3B%7D%20else%20if%20(window.exportcancel)%20%7BclearInterval(window.updatetimer)%3B%7Dtry%20%7Blet%20percentage%20%3D%20Math.floor((window.searchjsons.length%20%2F%20window.searches.length)%20*%20100)%3Bif%20(window.newui)%20%7Bwindow.listpage.getElementById('progressbar').value%20%3D%20percentage%3B%7D%20else%20%7Bwindow.listpage.getElementById('progressbar').style.width%20%3D%20percentage.toString()%20%2B%20'%25'%3B%7Dwindow.listpage.getElementById('progresspercentage').innerText%20%3D%20percentage.toString()%20%2B%20'%25'%3Bwindow.listpage.getElementById('progresstime').innerText%20%3D%20timer()%3B%7D%20catch%20%7B%7D%7Dfunction%20isScalar(value)%20%7Breturn%20(%2Fstring%7Cnumber%7Cboolean%2F).test(typeof%20value)%3B%7Dfunction%20flattenObject(obj%2C%20prefix%20%3D%20'')%20%7Blet%20result%20%3D%20%7B%7D%3Bfor%20(const%20%5Bkey%2C%20value%5D%20of%20Object.entries(obj))%20%7Bconst%20newKey%20%3D%20prefix%20%3F%20%60%24%7Bprefix%7D.%24%7Bkey%7D%60%20%3A%20key%3Bif%20(Array.isArray(value))%20%7Bif%20(value.every(isScalar))%20%7Bresult%5BnewKey%5D%20%3D%20value.join('%3B%20')%3B%7D%20else%20%7Bfor%20(let%20i%20%3D%200%3B%20i%20%3C%20value.length%3B%20i%2B%2B)%20%7Bconst%20tempResult%20%3D%20flattenObject(value%5Bi%5D%2C%20%60%24%7BnewKey%7D.%24%7Bi%7D%60)%3Bresult%20%3D%20%7B...result%2C...tempResult%7D%3B%7D%7D%7D%20else%20if%20(typeof%20value%20%3D%3D%3D%20'object'%20%26%26%20value%20!%3D%3D%20null)%20%7Bconst%20tempResult%20%3D%20flattenObject(value%2C%20newKey)%3Bresult%20%3D%20%7B...result%2C...tempResult%7D%3B%7D%20else%20%7Bresult%5BnewKey%5D%20%3D%20value%3B%7D%7Dreturn%20result%3B%7Dfunction%20getAllKeys(data)%20%7Blet%20keysSet%20%3D%20new%20Set()%3Bdata.forEach(item%20%3D%3E%20%7Bconst%20flattened%20%3D%20flattenObject(item)%3BObject.keys(flattened).forEach(key%20%3D%3E%20keysSet.add(key))%3B%7D)%3Breturn%20Array.from(keysSet)%3B%7Dfunction%20generateCSV(data)%20%7Bconst%20keys%20%3D%20getAllKeys(data)%3Bconst%20flattenedData%20%3D%20data.map(item%20%3D%3E%20flattenObject(item))%3Bconst%20csvRows%20%3D%20flattenedData.map(obj%20%3D%3E%20%7Breturn%20keys.map(key%20%3D%3E%20%60%22%24%7BString(obj%5Bkey%5D%20%3F%3F%20'').replace(%2F%22%2Fg%2C%20'%22%22')%7D%22%60).join('%2C')%3B%7D)%3Breturn%20%5Bkeys.join('%2C')%2C%20...csvRows%5D.join('%5Cn')%3B%7Dfunction%20downloadCSV()%20%7Blet%20blob%20%3D%20new%20Blob(%5B'%5Cufeff'%2C%20generateCSV(searchjsons)%5D%2C%20%7Btype%3A%20'text%2Fcsv'%7D)%3Blet%20objectUrl%20%3D%20URL.createObjectURL(blob)%3Blet%20dlink%20%3D%20document.createElement('a')%3Bdlink.setAttribute('href'%2C%20objectUrl)%3Bdlink.setAttribute('download'%2C%20'Saved%20searches.csv')%3Bdlink.click()%3B%7D%7D%7D%7D%7D)()
```

### Filter Analytics Entities From All Custodians
You run Name Normalization and suddenly the "All Custodians" field is polluted with tens of thousands of non-custodian entities without a way to separate them from Processing Custodians. Run this bookmarklet when in the Edit Search interface, and it will filter out all Analytics Entities when performing condition searches on any Entity field.
```js
let csrftoken = window.top.GetCsrfTokenFromPage();
let host = window.top.location.host;
let urlparams = new URLSearchParams(window.top.location.search);
let workspaceid = urlparams.get('AppID');
let listpage = document.getElementById('_ListPage').contentWindow;

fetch('https://' + host + '/Relativity.Rest/API/Relativity.Objects/workspace/' + workspaceid.toString() + '/object/queryslim', {
	'headers': {
		'accept': '*/*',
		'accept-language': 'en-US,en;q=0.9',
		'content-type': 'application/json',
		'sec-fetch-dest': 'empty',
		'sec-fetch-mode': 'cors',
		'sec-fetch-site': 'same-origin',
		'x-csrf-header': csrftoken
	},
	'referrerPolicy': 'strict-origin-when-cross-origin',
	'body': '{"request":{"objectType":{"artifactTypeID":7},"fields":[{"Name":"Artifact ID","ViewFieldID":1000467}],"condition":"","rowCondition":"(\'Name\' IN [\'Custodian – Processing\'])","sorts":[],"relationalField":null,"searchProviderCondition":null,"includeIdWindow":true,"convertNumberFieldValuesToString":true,"isAdHocQuery":false,"queryHint":null,"sampleParameters":null,"rankSortOrder":null,"cancellable":true},"start":1,"length":200}',
	'method': 'POST',
	'mode': 'cors',
	'credentials': 'include'
}).then(response => {
	return response.json();
}).then(jsn => {
	let multichoice = jsn['IDWindow'][0];
	fetch('https://' + host + '/Relativity.Rest/API/Relativity.Objects/workspace/' + workspaceid.toString() + '/object/queryslim', {
		'headers': {
			'accept': '*/*',
			'accept-language': 'en-US,en;q=0.9',
			'content-type': 'application/json',
			'sec-fetch-dest': 'empty',
			'sec-fetch-mode': 'cors',
			'sec-fetch-site': 'same-origin',
			'x-csrf-header': csrftoken
		},
		'referrerPolicy': 'strict-origin-when-cross-origin',
		'body': '{"request":{"objectType":{"artifactTypeID":25},"fields":[{"Name":"Artifact Type ID","ViewFieldID":1000742}],"condition":"","rowCondition":"(\'Name\' IN [\'Entity\'])","sorts":[],"relationalField":null,"searchProviderCondition":null,"includeIdWindow":true,"convertNumberFieldValuesToString":true,"isAdHocQuery":false,"queryHint":null,"sampleParameters":null,"rankSortOrder":null,"cancellable":true},"start":1,"length":200}',
		'method': 'POST',
		'mode': 'cors',
		'credentials': 'include'
	}).then(response => {
		return response.json();
	}).then(jsn => {
		let artifacttypeid = jsn['Objects'][0]['Values'][0];
		if (listpage.XMLHttpRequest.prototype.nativeSend === undefined) {
			listpage.XMLHttpRequest.prototype.nativeSend = listpage.XMLHttpRequest.prototype.send;
		}
		listpage.XMLHttpRequest.prototype.send = function(xhrdata) {
			try {
				let jsonobject = JSON.parse(xhrdata);
				if (jsonobject.request.objectType.artifactTypeID == artifacttypeid) {
					if (jsonobject.request.rowCondition == '') {
						jsonobject.request.rowCondition = "('Classification' INTERSECTS MULTICHOICE [" + multichoice.toString() + "])";
					} else {
						jsonobject.request.rowCondition = jsonobject.request.rowCondition.substring(0, jsonobject.request.rowCondition.length - 1) + " AND 'Classification' INTERSECTS MULTICHOICE [" + multichoice.toString() + "])";
					}
					this.nativeSend(JSON.stringify(jsonobject));
				} else {
					this.nativeSend(xhrdata);
				}
			} catch {
				this.nativeSend(xhrdata);
			}
		};
		try {
			listpage.document.querySelector('rwc-modal-layout.auto-width').querySelector('span[slot="actions"]').addEventListener('click', function(event) {
				listpage.XMLHttpRequest.prototype.send = listpage.XMLHttpRequest.prototype.nativeSend;
			});
		} catch {
			try {
				listpage.document.querySelector('rwc-modal-layout.medium').querySelector('span[slot="actions"]').addEventListener('click', function(event) {
					listpage.XMLHttpRequest.prototype.send = listpage.XMLHttpRequest.prototype.nativeSend;
				});
			} catch {
				try {
					listpage.document.querySelector('div.saved-search-viewer').querySelector('div.modal-footer').addEventListener('click', function(event) {
						listpage.XMLHttpRequest.prototype.send = listpage.XMLHttpRequest.prototype.nativeSend;
					});
				} catch {
					try {
						listpage.document.querySelector('div.modal-container').querySelector('div.modal-footer').addEventListener('click', function(event) {
							listpage.XMLHttpRequest.prototype.send = listpage.XMLHttpRequest.prototype.nativeSend;
						});
					} catch {
						listpage.XMLHttpRequest.prototype.send = listpage.XMLHttpRequest.prototype.nativeSend;
					}
				}
			}
		}
	});
});
```
To install this bookmarklet, create a new entry in your browser's Favorites Bar and set its URL to the following string:
```js
javascript:(function()%7Blet%20csrftoken%20%3D%20window.top.GetCsrfTokenFromPage()%3Blet%20host%20%3D%20window.top.location.host%3Blet%20urlparams%20%3D%20new%20URLSearchParams(window.top.location.search)%3Blet%20workspaceid%20%3D%20urlparams.get('AppID')%3Blet%20listpage%20%3D%20document.getElementById('_ListPage').contentWindow%3Bfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.Rest%2FAPI%2FRelativity.Objects%2Fworkspace%2F'%20%2B%20workspaceid.toString()%20%2B%20'%2Fobject%2Fqueryslim'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22request%22%3A%7B%22objectType%22%3A%7B%22artifactTypeID%22%3A7%7D%2C%22fields%22%3A%5B%7B%22Name%22%3A%22Artifact%20ID%22%2C%22ViewFieldID%22%3A1000467%7D%5D%2C%22condition%22%3A%22%22%2C%22rowCondition%22%3A%22(%5C'Name%5C'%20IN%20%5B%5C'Custodian%20%E2%80%93%20Processing%5C'%5D)%22%2C%22sorts%22%3A%5B%5D%2C%22relationalField%22%3Anull%2C%22searchProviderCondition%22%3Anull%2C%22includeIdWindow%22%3Atrue%2C%22convertNumberFieldValuesToString%22%3Atrue%2C%22isAdHocQuery%22%3Afalse%2C%22queryHint%22%3Anull%2C%22sampleParameters%22%3Anull%2C%22rankSortOrder%22%3Anull%2C%22cancellable%22%3Atrue%7D%2C%22start%22%3A1%2C%22length%22%3A200%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response%20%3D%3E%20%7Breturn%20response.json()%3B%7D).then(jsn%20%3D%3E%20%7Blet%20multichoice%20%3D%20jsn%5B'IDWindow'%5D%5B0%5D%3Bfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.Rest%2FAPI%2FRelativity.Objects%2Fworkspace%2F'%20%2B%20workspaceid.toString()%20%2B%20'%2Fobject%2Fqueryslim'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22request%22%3A%7B%22objectType%22%3A%7B%22artifactTypeID%22%3A25%7D%2C%22fields%22%3A%5B%7B%22Name%22%3A%22Artifact%20Type%20ID%22%2C%22ViewFieldID%22%3A1000742%7D%5D%2C%22condition%22%3A%22%22%2C%22rowCondition%22%3A%22(%5C'Name%5C'%20IN%20%5B%5C'Entity%5C'%5D)%22%2C%22sorts%22%3A%5B%5D%2C%22relationalField%22%3Anull%2C%22searchProviderCondition%22%3Anull%2C%22includeIdWindow%22%3Atrue%2C%22convertNumberFieldValuesToString%22%3Atrue%2C%22isAdHocQuery%22%3Afalse%2C%22queryHint%22%3Anull%2C%22sampleParameters%22%3Anull%2C%22rankSortOrder%22%3Anull%2C%22cancellable%22%3Atrue%7D%2C%22start%22%3A1%2C%22length%22%3A200%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response%20%3D%3E%20%7Breturn%20response.json()%3B%7D).then(jsn%20%3D%3E%20%7Blet%20artifacttypeid%20%3D%20jsn%5B'Objects'%5D%5B0%5D%5B'Values'%5D%5B0%5D%3Bif%20(listpage.XMLHttpRequest.prototype.nativeSend%20%3D%3D%3D%20undefined)%20%7Blistpage.XMLHttpRequest.prototype.nativeSend%20%3D%20listpage.XMLHttpRequest.prototype.send%3B%7Dlistpage.XMLHttpRequest.prototype.send%20%3D%20function(xhrdata)%20%7Btry%20%7Blet%20jsonobject%20%3D%20JSON.parse(xhrdata)%3Bif%20(jsonobject.request.objectType.artifactTypeID%20%3D%3D%20artifacttypeid)%20%7Bif%20(jsonobject.request.rowCondition%20%3D%3D%20'')%20%7Bjsonobject.request.rowCondition%20%3D%20%22('Classification'%20INTERSECTS%20MULTICHOICE%20%5B%22%20%2B%20multichoice.toString()%20%2B%20%22%5D)%22%3B%7D%20else%20%7Bjsonobject.request.rowCondition%20%3D%20jsonobject.request.rowCondition.substring(0%2C%20jsonobject.request.rowCondition.length%20-%201)%20%2B%20%22%20AND%20'Classification'%20INTERSECTS%20MULTICHOICE%20%5B%22%20%2B%20multichoice.toString()%20%2B%20%22%5D)%22%3B%7Dthis.nativeSend(JSON.stringify(jsonobject))%3B%7D%20else%20%7Bthis.nativeSend(xhrdata)%3B%7D%7D%20catch%20%7Bthis.nativeSend(xhrdata)%3B%7D%7D%3Btry%20%7Blistpage.document.querySelector('rwc-modal-layout.auto-width').querySelector('span%5Bslot%3D%22actions%22%5D').addEventListener('click'%2C%20function(event)%20%7Blistpage.XMLHttpRequest.prototype.send%20%3D%20listpage.XMLHttpRequest.prototype.nativeSend%3B%7D)%3B%7D%20catch%20%7Btry%20%7Blistpage.document.querySelector('rwc-modal-layout.medium').querySelector('span%5Bslot%3D%22actions%22%5D').addEventListener('click'%2C%20function(event)%20%7Blistpage.XMLHttpRequest.prototype.send%20%3D%20listpage.XMLHttpRequest.prototype.nativeSend%3B%7D)%3B%7D%20catch%20%7Btry%20%7Blistpage.document.querySelector('div.saved-search-viewer').querySelector('div.modal-footer').addEventListener('click'%2C%20function(event)%20%7Blistpage.XMLHttpRequest.prototype.send%20%3D%20listpage.XMLHttpRequest.prototype.nativeSend%3B%7D)%3B%7D%20catch%20%7Btry%20%7Blistpage.document.querySelector('div.modal-container').querySelector('div.modal-footer').addEventListener('click'%2C%20function(event)%20%7Blistpage.XMLHttpRequest.prototype.send%20%3D%20listpage.XMLHttpRequest.prototype.nativeSend%3B%7D)%3B%7D%20catch%20%7Blistpage.XMLHttpRequest.prototype.send%20%3D%20listpage.XMLHttpRequest.prototype.nativeSend%3B%7D%7D%7D%7D%7D)%3B%7D)%7D)()
```

### Download Native File
In some scenarios, Relativity does not let you download a native version of a document (i.e., due to the [ECA](https://help.relativity.com/RelativityOne/Content/Relativity/ECAv2_Overview.htm) app being installed in the workspace). This bookmarklet lets you download the currently displayed document's native file without going through the hassle of creating an Import/Export task.
```js
let host = window.top.location.host;
let urlparams = new URLSearchParams(window.top.location.search);
let workspaceid = urlparams.get('AppID');
let artifactid = urlparams.get('DocumentID');
let downloadurl = 'https://' + host + '/Relativity.REST/api/Relativity.Document//workspace/' + workspaceid.toString() + '/downloadnativefile/' + artifactid.toString();
let fname = document.getElementById('_ReviewInterface').contentWindow.document.getElementById('ri-document-action-title').innerText;
let fextension;

fetch(downloadurl).then(response => {
	return response.blob().then(blob => {
		let contentdisposition = response.headers.get('content-disposition');
		fextension = contentdisposition.slice(contentdisposition.lastIndexOf('.'));
		let objectUrl = URL.createObjectURL(blob);
		let dlink = document.createElement('a');
		dlink.setAttribute('href', objectUrl);
		dlink.setAttribute('download', fname + fextension);
		dlink.click();
	})
})
```
To install this bookmarklet, create a new entry in your browser's Favorites Bar and set its URL to the following string:
```js
javascript:(function()%7Blet%20host%20%3D%20window.top.location.host%3Blet%20urlparams%20%3D%20new%20URLSearchParams(window.top.location.search)%3Blet%20workspaceid%20%3D%20urlparams.get('AppID')%3Blet%20artifactid%20%3D%20urlparams.get('DocumentID')%3Blet%20downloadurl%20%3D%20'https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.REST%2Fapi%2FRelativity.Document%2F%2Fworkspace%2F'%20%2B%20workspaceid.toString()%20%2B%20'%2Fdownloadnativefile%2F'%20%2B%20artifactid.toString()%3Blet%20fname%20%3D%20document.getElementById('_ReviewInterface').contentWindow.document.getElementById('ri-document-action-title').innerText%3Blet%20fextension%3Bfetch(downloadurl).then(response%20%3D%3E%20%7Breturn%20response.blob().then(blob%20%3D%3E%20%7Blet%20contentdisposition%20%3D%20response.headers.get('content-disposition')%3Bfextension%20%3D%20contentdisposition.slice(contentdisposition.lastIndexOf('.'))%3Blet%20objectUrl%20%3D%20URL.createObjectURL(blob)%3Blet%20dlink%20%3D%20document.createElement('a')%3Bdlink.setAttribute('href'%2C%20objectUrl)%3Bdlink.setAttribute('download'%2C%20fname%20%2B%20fextension)%3Bdlink.click()%3B%7D)%7D)%7D)()
```

### Show Active Learning Projects
When the [ECA](https://help.relativity.com/RelativityOne/Content/Relativity/ECAv2_Overview.htm) app is installed in a workspace, RelativityOne hides the Active Learning tab. This bookmarklet lets you access this tab so you can quickly check past Active Learning Projects' statistics without the need to un-install and then re-install the ECA app.
```js
let csrftoken = window.top.GetCsrfTokenFromPage();
let host = window.top.location.host;
let urlparams = new URLSearchParams(window.top.location.search);
let workspaceid = urlparams.get('AppID');

fetch('https://' + host + '/Relativity.Rest/API/Relativity.Objects/workspace/' + workspaceid.toString() + '/object/queryslim', {
		'headers': {
			'accept': '*/*',
			'accept-language': 'en-US,en;q=0.9',
			'content-type': 'application/json',
			'sec-fetch-dest': 'empty',
			'sec-fetch-mode': 'cors',
			'sec-fetch-site': 'same-origin',
			'x-csrf-header': csrftoken
		},
		'referrerPolicy': 'strict-origin-when-cross-origin',
		'body': '{"request":{"objectType":{"artifactTypeID":23},"fields":[],"condition":"","rowCondition":"(\'Name\' IN [\'Active Learning Projects\'])","sorts":[],"relationalField":null,"searchProviderCondition":null,"includeIdWindow":true,"convertNumberFieldValuesToString":true,"isAdHocQuery":false,"queryHint":null,"sampleParameters":null,"rankSortOrder":null,"cancellable":true},"start":1,"length":200}',
		'method': 'POST',
		'mode': 'cors',
		'credentials': 'include'
}).then(response => {
	return response.json();
}).then(jsn => {
	fetch('https://' + host + '/Relativity.Rest/API/Relativity.Objects/workspace/' + workspaceid.toString() + '/object/queryslim', {
		'headers': {
			'accept': '*/*',
			'accept-language': 'en-US,en;q=0.9',
			'content-type': 'application/json',
			'sec-fetch-dest': 'empty',
			'sec-fetch-mode': 'cors',
			'sec-fetch-site': 'same-origin',
			'x-csrf-header': csrftoken
		},
		'referrerPolicy': 'strict-origin-when-cross-origin',
		'body': '{"request":{"objectType":{"artifactTypeID":25},"fields":[{\"Name\":\"Artifact Type ID\",\"ViewFieldID\":1000742}],"condition":"","rowCondition":"(\'Name\' IN [\'Active Learning Project\'])","sorts":[],"relationalField":null,"searchProviderCondition":null,"includeIdWindow":true,"convertNumberFieldValuesToString":true,"isAdHocQuery":false,"queryHint":null,"sampleParameters":null,"rankSortOrder":null,"cancellable":true},"start":1,"length":200}',
		'method': 'POST',
		'mode': 'cors',
		'credentials': 'include'
	}).then(response2 => {
		return response2.json();
	}).then(jsn2 => {
		window.location = 'https://' + host + '/Relativity/RelativityInternal.aspx?AppID=' + workspaceid.toString() + '&ArtifactTypeID=' + jsn2['Objects'][0]['Values'][0].toString() + '&Mode=ListPage&SelectedTab=' + jsn['IDWindow'][0].toString();
	});
});
```
To install this bookmarklet, create a new entry in your browser's Favorites Bar and set its URL to the following string:
```js
javascript:(function()%7Blet%20csrftoken%20%3D%20window.top.GetCsrfTokenFromPage()%3Blet%20host%20%3D%20window.top.location.host%3Blet%20urlparams%20%3D%20new%20URLSearchParams(window.top.location.search)%3Blet%20workspaceid%20%3D%20urlparams.get('AppID')%3Bfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.Rest%2FAPI%2FRelativity.Objects%2Fworkspace%2F'%20%2B%20workspaceid.toString()%20%2B%20'%2Fobject%2Fqueryslim'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22request%22%3A%7B%22objectType%22%3A%7B%22artifactTypeID%22%3A23%7D%2C%22fields%22%3A%5B%5D%2C%22condition%22%3A%22%22%2C%22rowCondition%22%3A%22(%5C'Name%5C'%20IN%20%5B%5C'Active%20Learning%20Projects%5C'%5D)%22%2C%22sorts%22%3A%5B%5D%2C%22relationalField%22%3Anull%2C%22searchProviderCondition%22%3Anull%2C%22includeIdWindow%22%3Atrue%2C%22convertNumberFieldValuesToString%22%3Atrue%2C%22isAdHocQuery%22%3Afalse%2C%22queryHint%22%3Anull%2C%22sampleParameters%22%3Anull%2C%22rankSortOrder%22%3Anull%2C%22cancellable%22%3Atrue%7D%2C%22start%22%3A1%2C%22length%22%3A200%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response%20%3D%3E%20%7Breturn%20response.json()%3B%7D).then(jsn%20%3D%3E%20%7Bfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.Rest%2FAPI%2FRelativity.Objects%2Fworkspace%2F'%20%2B%20workspaceid.toString()%20%2B%20'%2Fobject%2Fqueryslim'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22request%22%3A%7B%22objectType%22%3A%7B%22artifactTypeID%22%3A25%7D%2C%22fields%22%3A%5B%7B%5C%22Name%5C%22%3A%5C%22Artifact%20Type%20ID%5C%22%2C%5C%22ViewFieldID%5C%22%3A1000742%7D%5D%2C%22condition%22%3A%22%22%2C%22rowCondition%22%3A%22(%5C'Name%5C'%20IN%20%5B%5C'Active%20Learning%20Project%5C'%5D)%22%2C%22sorts%22%3A%5B%5D%2C%22relationalField%22%3Anull%2C%22searchProviderCondition%22%3Anull%2C%22includeIdWindow%22%3Atrue%2C%22convertNumberFieldValuesToString%22%3Atrue%2C%22isAdHocQuery%22%3Afalse%2C%22queryHint%22%3Anull%2C%22sampleParameters%22%3Anull%2C%22rankSortOrder%22%3Anull%2C%22cancellable%22%3Atrue%7D%2C%22start%22%3A1%2C%22length%22%3A200%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response2%20%3D%3E%20%7Breturn%20response2.json()%3B%7D).then(jsn2%20%3D%3E%20%7Bwindow.location%20%3D%20'https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity%2FRelativityInternal.aspx%3FAppID%3D'%20%2B%20workspaceid.toString()%20%2B%20'%26ArtifactTypeID%3D'%20%2B%20jsn2%5B'Objects'%5D%5B0%5D%5B'Values'%5D%5B0%5D.toString()%20%2B%20'%26Mode%3DListPage%26SelectedTab%3D'%20%2B%20jsn%5B'IDWindow'%5D%5B0%5D.toString()%3B%7D)%3B%7D)%7D)()
```

### Show Batch Sets
When the [ECA](https://help.relativity.com/RelativityOne/Content/Relativity/ECAv2_Overview.htm) app is installed in a workspace, RelativityOne hides the Batch Sets tab. This bookmarklet lets you access this tab so you can quickly check past Batch Sets without the need to un-install and then re-install the ECA app.
```js
let csrftoken = window.top.GetCsrfTokenFromPage();
let host = window.top.location.host;
let urlparams = new URLSearchParams(window.top.location.search);
let workspaceid = urlparams.get('AppID');

fetch('https://' + host + '/Relativity.Rest/API/Relativity.Objects/workspace/' + workspaceid.toString() + '/object/queryslim', {
		'headers': {
			'accept': '*/*',
			'accept-language': 'en-US,en;q=0.9',
			'content-type': 'application/json',
			'sec-fetch-dest': 'empty',
			'sec-fetch-mode': 'cors',
			'sec-fetch-site': 'same-origin',
			'x-csrf-header': csrftoken
		},
		'referrerPolicy': 'strict-origin-when-cross-origin',
		'body': '{"request":{"objectType":{"artifactTypeID":23},"fields":[],"condition":"","rowCondition":"(\'Name\' IN [\'Batch Sets\'])","sorts":[],"relationalField":null,"searchProviderCondition":null,"includeIdWindow":true,"convertNumberFieldValuesToString":true,"isAdHocQuery":false,"queryHint":null,"sampleParameters":null,"rankSortOrder":null,"cancellable":true},"start":1,"length":200}',
		'method': 'POST',
		'mode': 'cors',
		'credentials': 'include'
}).then(response => {
	return response.json();
}).then(jsn => {
	fetch('https://' + host + '/Relativity.Rest/API/Relativity.Objects/workspace/' + workspaceid.toString() + '/object/queryslim', {
		'headers': {
			'accept': '*/*',
			'accept-language': 'en-US,en;q=0.9',
			'content-type': 'application/json',
			'sec-fetch-dest': 'empty',
			'sec-fetch-mode': 'cors',
			'sec-fetch-site': 'same-origin',
			'x-csrf-header': csrftoken
		},
		'referrerPolicy': 'strict-origin-when-cross-origin',
		'body': '{"request":{"objectType":{"artifactTypeID":25},"fields":[{\"Name\":\"Artifact Type ID\",\"ViewFieldID\":1000742}],"condition":"","rowCondition":"(\'Name\' IN [\'Batch Set\'])","sorts":[],"relationalField":null,"searchProviderCondition":null,"includeIdWindow":true,"convertNumberFieldValuesToString":true,"isAdHocQuery":false,"queryHint":null,"sampleParameters":null,"rankSortOrder":null,"cancellable":true},"start":1,"length":200}',
		'method': 'POST',
		'mode': 'cors',
		'credentials': 'include'
	}).then(response2 => {
		return response2.json();
	}).then(jsn2 => {
		window.location = 'https://' + host + '/Relativity/RelativityInternal.aspx?AppID=' + workspaceid.toString() + '&ArtifactTypeID=' + jsn2['Objects'][0]['Values'][0].toString() + '&Mode=ListPage&SelectedTab=' + jsn['IDWindow'][0].toString();
	});
});
```
To install this bookmarklet, create a new entry in your browser's Favorites Bar and set its URL to the following string:
```js
javascript:(function()%7Blet%20csrftoken%20%3D%20window.top.GetCsrfTokenFromPage()%3Blet%20host%20%3D%20window.top.location.host%3Blet%20urlparams%20%3D%20new%20URLSearchParams(window.top.location.search)%3Blet%20workspaceid%20%3D%20urlparams.get('AppID')%3Bfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.Rest%2FAPI%2FRelativity.Objects%2Fworkspace%2F'%20%2B%20workspaceid.toString()%20%2B%20'%2Fobject%2Fqueryslim'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22request%22%3A%7B%22objectType%22%3A%7B%22artifactTypeID%22%3A23%7D%2C%22fields%22%3A%5B%5D%2C%22condition%22%3A%22%22%2C%22rowCondition%22%3A%22(%5C'Name%5C'%20IN%20%5B%5C'Batch%20Sets%5C'%5D)%22%2C%22sorts%22%3A%5B%5D%2C%22relationalField%22%3Anull%2C%22searchProviderCondition%22%3Anull%2C%22includeIdWindow%22%3Atrue%2C%22convertNumberFieldValuesToString%22%3Atrue%2C%22isAdHocQuery%22%3Afalse%2C%22queryHint%22%3Anull%2C%22sampleParameters%22%3Anull%2C%22rankSortOrder%22%3Anull%2C%22cancellable%22%3Atrue%7D%2C%22start%22%3A1%2C%22length%22%3A200%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response%20%3D%3E%20%7Breturn%20response.json()%3B%7D).then(jsn%20%3D%3E%20%7Bfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.Rest%2FAPI%2FRelativity.Objects%2Fworkspace%2F'%20%2B%20workspaceid.toString()%20%2B%20'%2Fobject%2Fqueryslim'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22request%22%3A%7B%22objectType%22%3A%7B%22artifactTypeID%22%3A25%7D%2C%22fields%22%3A%5B%7B%5C%22Name%5C%22%3A%5C%22Artifact%20Type%20ID%5C%22%2C%5C%22ViewFieldID%5C%22%3A1000742%7D%5D%2C%22condition%22%3A%22%22%2C%22rowCondition%22%3A%22(%5C'Name%5C'%20IN%20%5B%5C'Batch%20Set%5C'%5D)%22%2C%22sorts%22%3A%5B%5D%2C%22relationalField%22%3Anull%2C%22searchProviderCondition%22%3Anull%2C%22includeIdWindow%22%3Atrue%2C%22convertNumberFieldValuesToString%22%3Atrue%2C%22isAdHocQuery%22%3Afalse%2C%22queryHint%22%3Anull%2C%22sampleParameters%22%3Anull%2C%22rankSortOrder%22%3Anull%2C%22cancellable%22%3Atrue%7D%2C%22start%22%3A1%2C%22length%22%3A200%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response2%20%3D%3E%20%7Breturn%20response2.json()%3B%7D).then(jsn2%20%3D%3E%20%7Bwindow.location%20%3D%20'https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity%2FRelativityInternal.aspx%3FAppID%3D'%20%2B%20workspaceid.toString()%20%2B%20'%26ArtifactTypeID%3D'%20%2B%20jsn2%5B'Objects'%5D%5B0%5D%5B'Values'%5D%5B0%5D.toString()%20%2B%20'%26Mode%3DListPage%26SelectedTab%3D'%20%2B%20jsn%5B'IDWindow'%5D%5B0%5D.toString()%3B%7D)%3B%7D)%7D)()
```
