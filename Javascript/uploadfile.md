   
   ``` html
   <form>
   <input type="File" id="inputFile"/>
   </from>
   ```
   
   ``` js
   $(document).ready(function () {
        /*
        * Trigger a file open dialog and read local file, then read and log the file contents
        */


        var fileInput = $("#inputFile")[0];

        fileInput.addEventListener('change', function () {

            if (fileInput.files.length > 0) {

                var file = fileInput.files[0];

                if (file.name.match(/\.(xml|tpl)$/)) {
                    var reader = new FileReader();
                    var strContent = "";
                    reader.onload = function () {
                        strContent = reader.result;
                        console.log(strContent);
                        document.getElementById("fileContent").value = strContent;

                    };

                    reader.readAsText(file);
                } else {
                    alert("File not supported, .xml or .tpl files only");
                }
            }

        });
        
```
