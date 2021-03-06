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

views/layouts/app.blade.php
```
@yield('js')
```

views/questions/create.blade.php
```
<div class="form-group">
    <select class="js-example-basic-multiple form-control" multiple="multiple">
        <option value="AL">Alabama</option>
        <option value="WY">Wyoming</option>
    </select>
</div>

@section('js')
    <!-- 实例化编辑器 -->
    <script type="text/javascript">
        var ue = UE.getEditor('container', {
            toolbars: [
                ['bold', 'italic', 'underline', 'strikethrough', 'blockquote', 'insertunorderedlist', 'insertorderedlist', 'justifyleft','justifycenter', 'justifyright',  'link', 'insertimage', 'fullscreen']
            ],
            elementPathEnabled: false,
            enableContextMenu: false,
            autoClearEmptyNode:true,
            wordCount:false,
            imagePopup:false,
            autotypeset:{ indent: true,imageBlockLine: 'center' }
        });
        ue.ready(function() {
            ue.execCommand('serverparam', '_token', '{{ csrf_token() }}'); // 设置 CSRF token.
        });

        $(".js-example-basic-multiple").select2();
    </script>
@stop
```

gulpfile.js
```
mix.version(['js/app.js', 'css/app.css']);
```

views/layouts/app.blade.php
```
<link href="{{ elixir('css/app.css') }}" rel="stylesheet">
<script src="{{ elixir('js/app.js') }}"></script>
```

gulp

查看下拉框效果

database/factories/ModelFactory.php
```
/** @var \Illuminate\Database\Eloquent\Factory $factory */
$factory->define(App\Topic::class, function (Faker\Generator $faker) {
    return [
        'name' => $faker->word,
        'bio' => $faker->paragraph,
        'questions_count' => 1
    ];
});
```

php artisan tinker
namespace App;
factory(Topic::class, 11)->create()

将生成的测试数据中的name字段更改成如下：
![](image/screenshot_1491490093700.png)

views/questions/create.blade.php
```
<div class="form-group">
    <select name="topics[]" class="js-example-placeholder-multiple js-data-example-ajax form-control" multiple="multiple"></select>
</div>

@section('js')
    <!-- 实例化编辑器 -->
    <script type="text/javascript">
        var ue = UE.getEditor('container', {
            toolbars: [
                ['bold', 'italic', 'underline', 'strikethrough', 'blockquote', 'insertunorderedlist', 'insertorderedlist', 'justifyleft','justifycenter', 'justifyright',  'link', 'insertimage', 'fullscreen']
            ],
            elementPathEnabled: false,
            enableContextMenu: false,
            autoClearEmptyNode:true,
            wordCount:false,
            imagePopup:false,
            autotypeset:{ indent: true,imageBlockLine: 'center' }
        });
        ue.ready(function() {
            ue.execCommand('serverparam', '_token', '{{ csrf_token() }}'); // 设置 CSRF token.
        });

        $(function(){
            function formatTopic (topic) {
                return "<div class='select2-result-repository clearfix'>" +
                "<div class='select2-result-repository__meta'>" +
                "<div class='select2-result-repository__title'>" +
                topic.name ? topic.name : "Laravel"   +
                "</div></div></div>";
            }
            function formatTopicSelection (topic) {
                return topic.name || topic.text;
            }
            $(".js-example-placeholder-multiple").select2({
                tags: true,
                placeholder: '选择相关话题',
                minimumInputLength: 2,
                ajax: {
                    url: '/api/topics',
                    dataType: 'json',
                    delay: 250,
                    data: function (params) {
                        return {
                            q: params.term
                        };
                    },
                    processResults: function (data, params) {
                        return {
                            results: data
                        };
                    },
                    cache: true
                },
                templateResult: formatTopic,
                templateSelection: formatTopicSelection,
                escapeMarkup: function (markup) { return markup; }
            });
        });
    </script>
@stop
```

api.php
```
Route::get('/topics', function (Request $request) {
    $topics = \App\Topic::select(['id', 'name'])->where('name', 'like', '%'.$request->query('q').'%')->get();
    return $topics;
})->middleware('api');
```

测试api
![](image/screenshot_1491489297815.png)

QuestionController.php
```
/**
 * @param StoreQuestionRequest $request
 * @return \Illuminate\Http\RedirectResponse
 */
public function store(StoreQuestionRequest $request)
{
    dd($request->get('topics'));
    $data = [
        'title' => $request->get('title'),
        'body' => $request->get('body'),
        'user_id' => Auth::id()
    ];

    $question = Question::create($data);

    return redirect()->route('question.show', [$question->id]);
}
```

![](image/screenshot_1491489673199.png)

![](image/screenshot_1491489426909.png)