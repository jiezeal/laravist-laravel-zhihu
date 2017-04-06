#使用 Select2 优化话题选择

>http://select2.github.io/

cd resources/assets/css
```
wget https://cdnjs.cloudflare.com/ajax/libs/select2/4.0.3/css/select2.min.css
```

cd resources/assets/js
```
wget https://cdnjs.cloudflare.com/ajax/libs/select2/4.0.3/js/select2.min.js
```

resources/assets/js/bootstrap.js
```
require('./select2.min');
```

resources/assets/sass/app.scss
```
@import "../css/select2.min";
```

gulp

http://select2.github.io/examples.html

![](image/screenshot_1491486269379.png)

views/questions/create.blade.php
```
<div class="form-group">
    <select class="js-example-basic-multiple" multiple="multiple">
        <option value="AL">Alabama</option>
        <option value="WY">Wyoming</option>
    </select>
</div>
```




