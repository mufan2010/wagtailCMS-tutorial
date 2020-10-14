# 文章列表页分页

可以参考Django官方文档
https://docs.djangoproject.com/zh-hans/3.1/topics/pagination/

## 修改模型文件

Django提供了一个 Paginator 类来帮助我们管理分页数据。：

下面的示例表示一个典型的博客文章：

```python
from django.core.paginator import Paginator
from django.core.paginator import EmptyPage
from django.core.paginator import PageNotAnInteger
from hello.models import Topic

class BlogIndexPage(Page):
    intro = RichTextField(blank=True)

    def get_context(self, request):
        limit = 3  # 每页显示的记录数
        context = super().get_context(request) 
        blogpages = self.get_children().live().order_by('-first_published_at') # 获取内容
        paginator = Paginator(blogpages, limit) # 实例化一个分页对象
        page = request.GET.get('page') # 获取页码
    
        try:
            blogpages = paginator.page(page) # 获取某页对应的记录
        except PageNotAnInteger: # 如果页码不是个整数
            blogpages = paginator.page(1) # 取第一页的记录
        except EmptyPage: # 如果页码太大，没有相应的记录
            blogpages = paginator.page(paginator.num_pages) # 取最后一页的记录

        # 让变量在模板上可用
        context['blogpages'] = blogpages
        return context

    content_panels = Page.content_panels + [
        FieldPanel('intro', classname="full")
    ]

```
## 修改页面模板文件
模板中的相关操作

```html

        <nav aria-label="Page navigation">
            <ul class="pagination justify-content-center">
                {% if blogpages.has_previous %}
                    <li class="page-item">
                        
                          {# blogpages.paginator.number_pages 返回总页数 #}

                        <a class="page-link" href="?page={{ blogpages.previous_page_number }}">&laquo;</a>
                    </li>
                {% endif %}
                  {# blogpages.paginator.page_range 这个函数返回包含一个所有页码数的 range 对象 #}
                  {# 即 range(1, blogpages.paginator.num_pages + 1) #}
                
                {% for page_num in blogpages.paginator.page_range %}
                    <li {% if page_num == blogpages.number %} class="active page-item"{% endif %}>
                        <a class="page-link" href="?page={{ page_num }}">{{ page_num }}</a>
                    </li>
                {% endfor %}
                {% if blogpages.has_next %}
                    <li class="page-item">
                        <a class="page-link" href="?page={{ blogpages.next_page_number }}">&raquo;</a>
                    </li>
                {% endif %}
            </ul>
        </nav>



```


