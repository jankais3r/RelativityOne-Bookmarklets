# RelativityOne-Bookmarklets
A collection of bookmarklets for Relativity(One) administrators.


**Bookmarklets:**
- [Show Audit](#show-audit)
- [Edit Longtext Field](#edit-longtext-field)
- [Download Native File](#download-native-file)
- [Stop Inactivity Logout](#stop-inactivity-logout)


### Show Audit
Open a new window with an Audit Log of the currently displayed object. Useful for tracking down changes made to Saved Searches and similar objects.
```js
let host = window.top.location.host;
let urlparams = new URLSearchParams(window.top.location.search);
let workspaceid = urlparams.get('AppID');
let artifactid = urlparams.get('SelectedSearchArtifactID');
if (artifactid === null) {
	artifactid = urlparams.get('DocumentID');
	if (artifactid === null) {
		artifactid = urlparams.get('ArtifactID');
	}
}
if (artifactid === null) {
	alert('No ArtifactID field found');
} else {
	window.open('https://' + host + '/Relativity/custompages/270ea7b1-86c0-423c-ad52-0f2db09dec90/#/auditlist?WorkspaceID=' + workspaceid.toString() + '&ArtifactID=' + artifactid.toString() + '&DisplayLocation=Window', '_blank', 'popup');
}
```
To install this bookmarklet, create a new entry in your browser's Favorites Bar and set its URL to the following string:
```js
javascript:(function()%7Blet%20host%20%3D%20window.top.location.host%3Blet%20urlparams%20%3D%20new%20URLSearchParams(window.top.location.search)%3Blet%20workspaceid%20%3D%20urlparams.get('AppID')%3Blet%20artifactid%20%3D%20urlparams.get('SelectedSearchArtifactID')%3Bif%20(artifactid%20%3D%3D%3D%20null)%20%7Bartifactid%20%3D%20urlparams.get('DocumentID')%3Bif%20(artifactid%20%3D%3D%3D%20null)%20%7Bartifactid%20%3D%20urlparams.get('ArtifactID')%3B%7D%7Dif%20(artifactid%20%3D%3D%3D%20null)%20%7Balert('No%20ArtifactID%20field%20found')%3B%7D%20else%20%7Bwindow.open('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity%2Fcustompages%2F270ea7b1-86c0-423c-ad52-0f2db09dec90%2F%23%2Fauditlist%3FWorkspaceID%3D'%20%2B%20workspaceid.toString()%20%2B%20'%26ArtifactID%3D'%20%2B%20artifactid.toString()%20%2B%20'%26DisplayLocation%3DWindow'%2C%20'_blank'%2C%20'popup')%3B%7D%7D)()
```

### Edit Longtext Field
There are situations when you need to make manual adjustments to Extracted, OCR, or Transcribed Text of a specific document. This bookmarklet lets you edit any currently displayed longtext field straight from the document's Text Viewer without going through the hassle of creating an Import/Export task.
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
						'x-csrf-header': '\'' + csrftoken + '\''
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
javascript:(function()%7Blet%20csrftoken%20%3D%20window.top.GetCsrfTokenFromPage()%3Blet%20host%20%3D%20window.top.location.host%3Blet%20urlparams%20%3D%20new%20URLSearchParams(window.top.location.search)%3Blet%20workspaceid%20%3D%20urlparams.get('AppID')%3Blet%20documentid%20%3D%20urlparams.get('DocumentID')%3Blet%20reviewinterface%20%3D%20document.getElementById('_ReviewInterface').contentWindow.document%3Blet%20fieldname%20%3D%20reviewinterface.getElementById('ctrl-ri-tab-text-viewer').children%5B0%5D.children%5B0%5D.children%5B0%5D.children%5B0%5D.innerText%3Bif%20(urlparams.get('ViewerType')%20%3D%3D%20'text')%20%7Bfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.REST%2Fapi%2FRelativity-DocumentViewer%2Fv1%2FPrivateDocumentViewerServiceManager%2FGetReviewInterfaceData'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22request%22%3A%7B%22WorkspaceId%22%3A'%20%2B%20workspaceid.toString()%20%2B%20'%2C%22ViewerPreference%22%3A7%2C%22Options%22%3A%7B%22ClientId%22%3A%22Relativity.ReviewInterface.PrepareDocuments%22%7D%2C%22DocumentIds%22%3A%5B'%20%2B%20documentid.toString()%20%2B%20'%5D%7D%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response%20%3D%3E%20%7Breturn%20response.json()%3B%7D).then(jsn%20%3D%3E%20%7Blet%20LongTextFields%20%3D%20Object.values(jsn%5B'DocumentData'%5D)%5B0%5D%5B'LongTextVisibility'%5D%3Blet%20longtextfieldid%3Bfor%20(const%20id%20in%20LongTextFields)%20%7Bif%20(LongTextFields%5Bid%5D%5B'TextFieldName'%5D%20%3D%3D%20fieldname)%20%7Blongtextfieldid%20%3D%20LongTextFields%5Bid%5D%5B'FieldArtifactId'%5D%3B%7D%7Dfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.REST%2Fapi%2FRelativity-DocumentViewer%2Fv1%2FPrivateDocumentViewerServiceManager%2FGetTextViewerContent'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20csrftoken%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22request%22%3A%7B%22WorkspaceId%22%3A'%20%2B%20workspaceid.toString()%20%2B%20'%2C%22ArtifactId%22%3A'%20%2B%20documentid.toString()%20%2B%20'%2C%22LongTextFieldArtifactId%22%3A'%20%2B%20longtextfieldid.toString()%20%2B%20'%2C%22PageSize%22%3A999999999%2C%22PageIndex%22%3A0%7D%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response%20%3D%3E%20%7Breturn%20response.json()%3B%7D).then(jsn%20%3D%3E%20%7Blet%20oitpageblock%3Bfor%20(const%20oit%20in%20reviewinterface.getElementsByClassName('ri-text-viewer')%5B0%5D.getElementsByClassName('oit-pageblock'))%20%7Bif%20(reviewinterface.getElementsByClassName('oit-pageblock')%5Boit%5D.className%20%3D%3D%20'oit-pageblock')%20%7Boitpageblock%20%3D%20reviewinterface.getElementsByClassName('ri-text-viewer')%5B0%5D.getElementsByClassName('oit-pageblock')%5Boit%5D.parentNode%3Bbreak%3B%7D%7Dlet%20originalcontent%20%3D%20oitpageblock.innerHTML%3Boitpageblock.style.textAlign%20%3D%20'center'%3Boitpageblock.innerHTML%20%3D%20'%3Cdiv%20class%3D%22oit-pageblock%22%3E%3Cbutton%20id%3D%22cancelbutton%22%20class%3D%22rwa-button%20secondary%22%20style%3D%22padding%3A%203px%208px%3B%20line-height%3A%2017px%3B%20margin%3A%205px%3B%22%3ECancel%3C%2Fbutton%3E%3Cbutton%20id%3D%22savebutton%22%20class%3D%22rwa-button%20primary%22%20style%3D%22padding%3A%203px%208px%3B%20line-height%3A%2017px%3B%20margin%3A%205px%3B%22%3ESave%20edits%3C%2Fbutton%3E%3Cbr%3E%3Ctextarea%20id%3D%22extractedtextarea%22%20style%3D%22width%3A100%25%3B%22%3E%3C%2Ftextarea%3E%3C%2Fdiv%3E'%3Breviewinterface.getElementById('extractedtextarea').value%20%3D%20jsn%5B'LongTextContent'%5D%3Breviewinterface.getElementById('extractedtextarea').style.height%20%3D%20(reviewinterface.getElementById('extractedtextarea').scrollHeight%20%2B%2010)%20%2B%20'px'%3Breviewinterface.getElementById('cancelbutton').onclick%20%3D%20function()%20%7Boitpageblock.style.textAlign%20%3D%20''%3Boitpageblock.innerHTML%20%3D%20originalcontent%3B%7D%3Breviewinterface.getElementById('savebutton').onclick%20%3D%20function()%20%7Breviewinterface.getElementById('extractedtextarea').disabled%20%3D%20true%3Breviewinterface.getElementById('cancelbutton').disabled%20%3D%20true%3Breviewinterface.getElementById('savebutton').disabled%20%3D%20true%3Bfetch('https%3A%2F%2F'%20%2B%20host%20%2B%20'%2FRelativity.REST%2Fapi%2FRelativity.ObjectManager%2Fv1%2Fworkspace%2F'%20%2B%20workspaceid.toString()%20%2B%20'%2Fobject%2Fupdate'%2C%20%7B'headers'%3A%20%7B'accept'%3A%20'*%2F*'%2C'accept-language'%3A%20'en-US%2Cen%3Bq%3D0.9'%2C'content-type'%3A%20'application%2Fjson'%2C'sec-fetch-dest'%3A%20'empty'%2C'sec-fetch-mode'%3A%20'cors'%2C'sec-fetch-site'%3A%20'same-origin'%2C'x-csrf-header'%3A%20'%5C''%20%2B%20csrftoken%20%2B%20'%5C''%7D%2C'referrerPolicy'%3A%20'strict-origin-when-cross-origin'%2C'body'%3A%20'%7B%22request%22%3A%7B%22Object%22%3A%7B%22artifactId%22%3A'%20%2B%20documentid.toString()%20%2B%20'%7D%2C%22FieldValues%22%3A%5B%7B%22Field%22%3A%7B%22ArtifactID%22%3A'%20%2B%20longtextfieldid.toString()%20%2B%20'%7D%2C%22Value%22%3A'%20%2B%20JSON.stringify(reviewinterface.getElementById('extractedtextarea').value.replaceAll('%5Cn'%2C%20'%5Cr%5Cn'))%20%2B%20'%7D%5D%7D%2C%22OperationOptions%22%3A%7B%22CallingContext%22%3Anull%7D%7D'%2C'method'%3A%20'POST'%2C'mode'%3A%20'cors'%2C'credentials'%3A%20'include'%7D).then(response%20%3D%3E%20%7Blocation.reload()%3B%7D)%3B%7D%3B%7D)%7D)%7D%7D)()
```

### Download Native File
In some scenarios, Relativity's UI does not let you download a native version of a document (i.e., due to the ECA app being installed in the workspace). This bookmarklet lets you download the currently displayed document's native file without going through the hassle of creating an Import/Export task.
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

### Stop Inactivity Logout
We all know how counter-productive an unexpected logout from Relativity can be. This bookmarklet calls the `resetExpirationTimeout` function every 5 minutes, making sure your sessions does not expire due to inactivity. Technically, only the first line is necessary, the rest of the code changes color of some UI elements to show you the script has been activated.
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

