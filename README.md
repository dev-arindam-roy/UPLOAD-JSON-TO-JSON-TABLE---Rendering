# UPLOAD JSON TO JSON & TABLE - Rendering

### Check The Application

[https://dev-arindam-roy.github.io/UPLOAD-JSON-TO-JSON-TABLE---Rendering/](https://dev-arindam-roy.github.io/UPLOAD-JSON-TO-JSON-TABLE---Rendering/)


```js
/** SweetAlert2 loading */
const displayLoading = (timer = 3000, title = 'Please Wait...', text = "System Processing Your Request") => {
    Swal.fire({
        title: title,
        text: text,
        allowEscapeKey: false,
        allowOutsideClick: false,
        timer: timer,
        didOpen: () => {
            Swal.showLoading()
        }
    });
}

/** SweetAlert2 like toast */
const displayToast = () => {
    Swal.fire({
        position: 'top-end',
        icon: 'success',
        title: 'Its copied!',
        showConfirmButton: false,
        timer: 1000
    });
}

$("#frmx").validate({
    errorClass: 'onex-error',
    errorElement: 'div',
    rules: {
        json_file: {
            required: true,
            extension: 'json'
        }
    },
    messages: {
        json_file: {
            required: 'Please upload json file',
            extension: 'Only accept .json',
            accept: 'Only accept .json'
        }
    },
    errorPlacement: function (error, element) {
        if(element.hasClass('onex-select2')) {
            error.insertAfter(element.parent().find('span.select2-container'));
        } else {
            error.insertAfter(element);
        }
    },
    highlight: function (element) {
        $(element).removeClass('is-valid').addClass('is-invalid');
        $(element).parent().find('.onex-form-label').addClass('onex-error-label');
        if(element.type == 'select-one') {
            $(element).next('span.select2-container').addClass('select2-custom-error');
        }
    },
    unhighlight: function (element) {
        $(element).removeClass('is-invalid').addClass('is-valid');
        $(element).parent().find('.onex-form-label').removeClass('onex-error-label');
    },
    submitHandler: function (form) {
        uploadJsonFile();
        return false;
    }
});

/* Json To HTML Table Create */
const jsonTable = (objArr, renderId, appName = '') => {

    /* create table */
    let _table = document.createElement('table');
    _table.setAttribute('id', 'dataTable');
    _table.classList.add('table', 'table-bordered', 'table-striped', 'table-sm');

    if (objArr.length) {
        
        let firstObjKeys = Object.keys(objArr[0]) || [];
        let firstObjValues = Object.values(objArr[0]) || [];
        let firstObjKeyLength = parseInt(firstObjKeys.length);
        

        /* create thead & apped to table */
        let _thead = document.createElement('thead');
        _table.appendChild(_thead);

        if (appName !== '') {
            /* create heading row with colspan td */
            let _headingTr = _thead.insertRow();
            let _headingTd = _headingTr.insertCell(0);
            _headingTd.colSpan = firstObjKeyLength;
            _headingTd.innerHTML = `<strong>${appName}</strong> (${objArr.length})`;
        }

        objArr.map((value, index) => {

            if (index === 0) {
                /* add or insert tr within thead */
                let _headTr = _thead.insertRow();

                /* dynamically create th & append to above tr*/
                for (let i = 0; i < firstObjKeyLength; i++) {
                    let _headTh = document.createElement('th');
                    let _headThText = document.createTextNode(`${firstObjKeys[i].charAt(0).toUpperCase()}${firstObjKeys[i].slice(1)}`);
                    _headTh.appendChild(_headThText);
                    _headTr.appendChild(_headTh);
                }   
            } else {
                /* create tbody & append to table */
                let _tbody = document.createElement('tbody');
                    _table.appendChild(_tbody);

                let _tr = _tbody.insertRow();

                /* dynamically create td with object value & append to above tr in map loop*/
                for (let i = 0; i < firstObjKeyLength; i++) {
                    let _objKeys = Object.keys(value);
                    let _objVals = Object.values(value);
                    let _td = _tr.insertCell(i);
                    _td.innerHTML = `${_objVals[i]}`;
                }
            }
        });
        /* finally table render to html */
        document.getElementById(renderId).innerHTML = '';
        document.getElementById(renderId).appendChild(_table);
    } else {
        document.getElementById(renderId).innerHTML = `<h4>No Records Found!</h4>`;
    }
}

function actionButtons(jsonObj) {
    if(jsonObj.length) {
        $('#downloadJsonBtn').show();
        $('#downloadCsvBtn').show();
    } else {
        $('#downloadJsonBtn').hide();
        $('#downloadCsvBtn').hide();
    }
}

/*Json Content To CSV content*/
function jsonToCsv(jsonData) {
    let csv = '';
    // Get the headers
    let headers = Object.keys(jsonData[0]);
    csv += headers.join(',') + '\n';
    // Add the data
    jsonData.forEach(function (row) {
        let data = headers.map(header => row[header]).join(',');
        csv += data + '\n';
    });
    return csv;
}

let jsonFile = document.querySelector('#jsonFileUpload');

/*Get Json File Content From Uploader*/
async function uploadJsonFile() {
    displayLoading();
    let file = jsonFile.files[0];
    renderAsJson(file);
    renderAsTable(file);
}

async function renderAsTable(file) {
    let reader = new FileReader(file);
    reader.readAsText(file);
    reader.onload = async(e) => {
        let resultData = e.target.result;
        let jsonContent = await JSON.parse(resultData);
        console.log('table');
        console.log(jsonContent);
        jsonTable(jsonContent, 'renderTable');
        document.getElementById('tableCount').innerHTML = `(${jsonContent.length})`;
        actionButtons(jsonContent);
    }
}

async function renderAsJson(file) {
    let reader = new FileReader(file);
    reader.readAsText(file);
    reader.onload = async(e) => {
        let resultData = e.target.result;
        let jsonContent = await JSON.parse(resultData);
        console.log(jsonContent);
        $('#convertedJson').val(JSON.stringify(jsonContent, null, 4));
        $('#json-renderer').jsonViewer(jsonContent);
        document.getElementById('jsonCount').innerHTML = `(${jsonContent.length})`;
        actionButtons(jsonContent);
    }
}

/*Download as JSON*/
$('#downloadJsonBtn').on('click', function() {
    const anchor = document.createElement('a');
    anchor.href = 'data:application/json;charset=utf-8,' + encodeURIComponent($('#convertedJson').val());
    anchor.download = new Date().toJSON() + '.json' ;
    document.body.appendChild(anchor);
    anchor.click();
    document.body.removeChild(anchor);
});

/*Download as CSV*/
$('#downloadCsvBtn').on('click', function() {
    let content = jsonToCsv(JSON.parse($('#convertedJson').val()));
    let blob = new Blob([content], { type: 'text/csv' });
    let url = window.URL.createObjectURL(blob);
    let a = document.createElement('a');
    a.href = url;
    a.download = new Date().toJSON() + '.csv' ;
    document.body.appendChild(a);
    a.click();
});
```
