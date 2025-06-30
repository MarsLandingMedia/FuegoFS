# FuegoFS

FuegoFS is a lightweight file serving application for ServiceNow. It stores all of its file content assets in a custom table and exposes a REST endpoint so the files can be retrieved by path. This allows small assets or pages to be embedded directly within your instance.

---

## How It Works

1. **Files Stored in the FuegoFileService Table**  
   Records hold the file body, MIME type, and a unique path.
2. **REST Call to `/api/x_fuegofs/service`**  
   A GET request with the desired path (e.g., `?index.html`) is issued.
3. **`FuegoFrenzy` Script Include Finds the Record**  
   The path is matched to an active record and the content is returned.
4. **Response Includes MIME Type and Body**  
   The REST API sets the response content type and streams the file back to the caller.

---

## Features

- Simple REST interface for serving stored files
- Supports common MIME types such as HTML, CSS, JavaScript, images, and PDF
- Path-based lookup so multiple files can be managed in one table
- Script Include `FuegoFrenzy` can be called from other scripts

---

## Installation

1. Import the FuegoFS scoped application into ServiceNow.
2. Verify that the `FuegoFrenzy` script include and REST service are active.
3. Create records in **FuegoFileService** with the desired path, MIME type, and content.
4. Retrieve files using the REST endpoint:
   `GET https://your-instance.service-now.com/api/x_fuegofs/service?/view/hello.html`

---

## Example Script Usage

```javascript
var ff = new x_fuegofs.FuegoFrenzy();
var file = ff.FileServe("/view/hello.html");
// file.mimetype => "text/html"
// file.bdy      => content of the file
gs.info(file.bdy);
```

### To give you an idea of what to expect upon output, here's the HTML in the table record that comes with the install.
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello World Test</title>
</head>
<body>
    <h1>Hello, world!</h1>
</body>
</html>
```

---

## Disclaimers

**Legal and Use Notice**  
This project is licensed under the [GNU Affero General Public License](https://www.gnu.org/licenses/gpl-3.0.html) and is a product of **Mars Landing Media LLC**. It is provided solely for demonstration and educational purposes. No warranties, maintenance, or official support of any kind are provided. Use at your own risk.

**ServiceNow® Notice**  
ServiceNow is a registered trademark of ServiceNow, Inc. This project is not affiliated with, endorsed by, or sponsored by ServiceNow. Ensure usage complies with ServiceNow's Terms of Service.


**GlideRecordSecure Notice**  
This project does not enforce GlideRecordSecure and assumes a trusted development environment.
