# 文章列表页分页

可以参考Django官方文档
https://docs.djangoproject.com/zh-hans/3.1/topics/pagination/

## 修改模型文件

Django提供了一个 Paginator 类来帮助我们管理分页数据。：

下面的示例表示一个典型的博客文章：

```python
class BlogIndexPage(Page):
    intro = RichTextField(blank=True)

    def get_context(self, request):
        # Update context to include only published posts, ordered by reverse-chron
        context = super().get_context(request)
        blogpages = self.get_children().live().order_by('-first_published_at')

        paginator = Paginator(blogpages, 3) # Show 3 resources per page

        page = request.GET.get('page')
        try:
            blogpages = paginator.page(page)
        except PageNotAnInteger:
            # If page is not an integer, deliver first page.
            blogpages = paginator.page(1)
        except EmptyPage:
            # If page is out of range (e.g. 9999), deliver last page of results.
            blogpages = paginator.page(paginator.num_pages)

        # make the variable 'resources' available on the template
        context['blogpages'] = blogpages
        return context

    content_panels = Page.content_panels + [
        FieldPanel('intro', classname="full")
    ]

```
## 修改页面模板文件
采用Boostrap 4 的分页 Pagination 导航样式，编辑 blog_index_page.html ，内容如下：
```html
{% extends "base.html" %}

{% load wagtailcore_tags wagtailimages_tags %}

{% block body_class %}template-blogindexpage{% endblock %}

{% block content %}

        <h1 class="mt-2">{{ page.title }}</h1>

        <!-- <div class="intro">{{ page.intro|richtext }}</div> -->

        {% for post in blogpages %}
            {% with post=post.specific %}

                <div class="card mb-2">
                    <h4 class="card-header"><a href="{% pageurl post %}">{{ post.title }}</a></h4>
                    <div class="card-body">
                        <div class="row">
                            <div class="col-auto mr-auto">

                                {% if post.intro %}
                                    {{ post.intro|richtext }}
                                {% else %}
                                    {{ post.body|richtext|truncatewords_html:80 }}
                                {% endif %}

                                <a href="{% pageurl post %}" class="btn btn-primary mt-2 mb-2">阅读全文</a>
                            </div>
                            <div class="col-auto">
                                {% with post.main_image as main_image %}
                                    {% if main_image %}
                                        <a href="{% pageurl post %}">
                                            {% image main_image fill-160x100 %}
                                        </a>
                                    {% endif %}
                                {% endwith %}
                            </div>
                        </div>

                    </div>
                </div>

            {% endwith %}
        {% endfor %}

        <nav aria-label="Page navigation">
            <ul class="pagination justify-content-center">
                {% if blogpages.has_previous %}
                    <li class="page-item">
                        <a class="page-link" href="?page={{ blogpages.previous_page_number }}">&laquo;</a>
                    </li>
                {% endif %}
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

        <hr>

{% endblock %}

```


