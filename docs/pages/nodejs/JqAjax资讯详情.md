```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>天目里</title>
    <script src="http://code.jquery.com/jquery-2.1.1.min.js"></script>
    <style>
      body {
        font-size: 16;
        padding: 5px 17px;
        color: #656565;
      }
      img {
        margin-top: 5px;
      }
    </style>
  </head>
  <body id="content"></body>
  <script>
    $(document).ready(function () {
      var param = GetRequest();
      $.ajax({
        type: "get",
        url:
          "http://39.170.11.108:9100/ic/web/information/client/information/getById",
        data: { id: param.id },
        headers: {
          "Content-Type": "application/json;charset=utf8",
          token: param.token,
        },
        dataType: "json",
        success: function (response) {
          $("#content").html(response.result.informationText);
        },
      });
    });

    function GetRequest() {
      var url = location.search; //获取url中"?"符后的字串
      var theRequest = new Object();
      if (url.indexOf("?") != -1) {
        var str = url.substr(1);
        strs = str.split("&");
        for (var i = 0; i < strs.length; i++) {
          theRequest[strs[i].split("=")[0]] = unescape(strs[i].split("=")[1]);
        }
      }
      return theRequest;
    }
  </script>
</html>

```