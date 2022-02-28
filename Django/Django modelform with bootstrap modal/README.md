# Django modelform with bootstrap modal

Created: February 1, 2022 5:22 PM
Tags: Django, HTML

前端使用 bootstrap modal 彈窗表單

後端使用 Django modelform 生成表單

後端:

1. 視圖 [views.py](http://views.py) 
    
    ```python
    # views.py
    
    ## 主頁
    class TestView(CustomPagination, TemplateView):
        template_name = 'test/test.html'
        success_message = ''
    
        @method_decorator(gzip_page)
        @method_decorator(my_login_required)
        @method_decorator(csrf_protect)
    		# 使用 get 回傳 orm 檢索結果
        def get(self, request, *args, **kwargs):
            context = self.get_context_data(**kwargs)
            results = Test.objects.filter(uid_id=request.session['user_id'])
            context['results'] = results
            return self.render_to_response(context)
    
    ## 彈窗視窗頁面
    @method_decorator(never_cache, name='dispatch')
    class TestEditFormView(CustomPagination, TemplateView):
        template_name = 'test/test_edit.html'
        
        @method_decorator(gzip_page)
        @method_decorator(my_login_required)
        @method_decorator(csrf_protect)
        # get 初次載入表單，需要 get 才能直接取到 instance
        def get(self, request, *args, **kwargs):
            instance = Test.objects.get(id=request.GET['test_id'])
            form = TestForm(request.POST or None, instance=instance)
            context = self.get_context_data(**kwargs)
            context['form'] = form
            return self.render_to_response(context)
    
        @method_decorator(gzip_page)
        @method_decorator(my_login_required)
        @method_decorator(csrf_protect)
        # post 提交修改後的表單
        def post(self, request, *args, **kwargs):
            instance = Test.objects.get(id=request.POST['test_id'])
            form = TestForm(request.POST or None, instance=instance)
            if form.is_valid():
                # 儲存更新表單
                form.save()
            # 回至主頁
            return redirect((reverse('test:test')))
    ```
    
2. 表單 form[s.py](http://views.py) 
    
    ```python
    class TestForm(forms.ModelForm):
    
        class Meta:
            model = Test
            # fields = '__all__'
            fields = ('id', 'title', 'email', 'subject', 'searchquery_show', 'stat', 'test_date', 'expire_date')
            # 套用 bootstrap 語法
            widgets = {
                'title': forms.TextInput(attrs={'class': 'form-control'}),
                'email': forms.TextInput(attrs={'class': 'form-control'}),
                'subject': forms.TextInput(attrs={'class': 'form-control'}),
                'test_date': forms.TextInput(attrs={'class': 'form-control'}),
                'searchquery_show': forms.TextInput(attrs={
                    'class': 'form-control',
                    'readonly': 'readonly'
                }),
                'expire_date':widgets.SelectDateWidget()
            }
            # locale 翻譯
            labels = {
                "title": _("Title  "),
                "email": _("Email"),
                "subject": _("Subject"),
                "test_date": _("TEST execution cycle (in days)"),
                "searchquery_show": _("Search strategy"),
                "stat": _("Do you want to be notified if there is no new information?"),
                "expire_date": _("Test expire date"),
            }
    ```
    
3. 路由 [urls.py](http://urls.py) 
    
    ```python
    url(r'^test/$', views.TestView.as_view(), name='test'),
    url(r'^test_edit/$', views.TestEditFormView.as_view(), name='test_edit'),
    ```
    

前端:

1.  test.html
    
    ```python
    
    <table class="rwd-table m-0">
        <tr>
            <th width="50">{% trans 'ID' %}</th>
            <th width="300">{% trans 'Search test' %}</th>
            <th width="80">{% trans 'Edit New Noticed' %}</th>
        </tr>
        {% for item in results %}
         <tr>
            <td aria-label="編號" data-th="編號">
                <label class="m-0 lh-1">{{ forloop.counter }}<input type="checkbox" name="hsel" value="{{ item.id }}" class="m-0 lh-1"><input type="hidden" value="{{ item.searchquery }}"></label>
            </td>
            <td aria-label="檢索test" data-th="檢索test" name="a{{ forloop.counter }}box">
                <span href="" name="a{{ forloop.counter }}a">{% if item.searchquery_show %}{{ item.searchquery_show }}{% else %}{{ item.searchquery }}{% endif %}</span>
            </td>
            <td aria-label="內容" data-th="內容">
                <!-- Button trigger modal -->
                <button type="button" class="btn btn-default"  onclick='modalAction("#testModalCenter", "{% url 'test:test_edit' %}", "{{ item.id }}")'>
                    {% trans 'Edit New Noticed' %}
                </button>
            </td>
        </tr>
        {% endfor %}
    </table>
    <div class="modal fade" id="testModalCenter" tabindex="-1" role="dialog" aria-labelledby="testModalCenterTitle" aria-hidden="true"></div>
    <script type="text/javascript">
    // js code 
    
    function modalAction(result, url, id) {
        $.ajaxSetup({data:{csrfmiddlewaretoken:'{{ csrf_token }}'},});
        $.ajax({url:url,async :false,data:{"test_id":id},type:"GET",success:function (data) { 
            $(result).modal('show');
            $(result).html(data);
        }})
    };
    </script>
    ```
    
2. test_edit.html
    
    ```html
    {% load static i18n %}
    <div class="modal-dialog modal-lg" role="document">
        <div class="modal-content">
            <div class="modal-header">
                <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span
                        aria-hidden="true">&times;</span></button>
                <h4 class="modal-title" id="modalLabel_change">{% trans 'Edit New Noticed' %}</h4>
            </div>
            <form action="{% url 'thesis:test_edit' %}" method="POST" id="test_form">
                {% csrf_token %}
                <div class="modal-body">
                    <div class="form-horizontal">
                        <div class="col-xs-12 clearfix">
                        <!-- 傳 id 值至後端判斷 -->
                        <input type='hidden' value='{{form.instance.id}}' name='test_id'>
                            {% for field in form %}
                                <div class="form-group">
                                    <label class="col-xs-3 text-right {% if field.field.required %} field-required{% endif %}" id="{{ field.auto_id }}" name="{{ field.name }}">{{field.label}}</label>
                                    <div class="col-xs-9">{{field}}</div>
                                </div>
                            {% endfor %}
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="submit" class="btn btn-primary">{% trans 'Confirm' %}</button>
                    <button type="button" class="btn btn-default" data-dismiss="modal">{% trans 'Cancel' %}</button>
                    
                </div>
        </div>
    </div>
    <!-- 使用 validate 驗證表單 -->
    <script src='{% static "js/jquery.validate.min.js" %}'></script>
    <script>
        $().ready(function() {
            $("#test_form").validate({
                rules: {
                    title: {
                        required: true
                    },
                    email: {
                        required: true,
                        email: true
                    }, 
            
                },
                //錯誤提示
                messages: {
                    title: "{% trans 'Please input title' %}",
                    email: {
                        required: "{% trans 'Please input Email address' %}",
                        email: "{% trans 'Please check Email address format' %}",
                    },
                },
            });
        });
    
    </script>
    
    <style>
    .field-label {
        font-weight: normal;
    }
    .field-helptext {
        font-size: 12px;
    }
    .field-error {
        font-size: 14px;
        color: red;
    }
    .field-required {
        font-weight: bold;
    }
    .field-required:after {
        content: " *";
    }
    </style>
    ```