# 作者：single430  时间：2016.10.11
<html>
<body>
<h2>一：利用Django自带用户认证系统实现用户的登录注册注销</h2>
  online/views.py
  
    from django.shortcuts import render, redirect, render_to_response<br/>
    from django.template.context import RequestContext<br/>

    from online.models import MyUser<br/>
    from django.utils import timezone<br/>
    from blogs.views import my_pagination<br/>
    from blogs.models import BlogsManage<br/>
    from django.contrib import auth<br/>
    from blogs.forms import LoginForm<br/>
    def log_success(request):
      if request.method == 'GET':
          form = LoginForm()
          return redirect('/online/', RequestContext(request, {'form': form, }))
      else:
          form = LoginForm(request.POST)
          if form.is_valid():
              print 'form.is_valid(): ', form.is_valid()
              # 获取表单信息
              username = request.POST.get('username', '')
              password = request.POST.get('password', '')
              user = auth.authenticate(username=username, password=password)
              print username, password, user
              if user is not None and user.is_active:
                  auth.login(request, user)
                  # print username, password
                  # 对比
                  userinfo = MyUser.objects.filter(username=username, password=password)
                  # 判断用户是否存在
                  if bool(userinfo):
                      # 查看用户是否有文章
                      all_objects = BlogsManage.objects.filter(author=username)
                      objects, page_range = my_pagination(request, all_objects)
                      return redirect('/blogs/%s' % username, {'userinfo': userinfo, 'objects': objects, 'page_range': page_range})
                  else:
                      return render(request, 'online/login.html', {'error_message': "用户名或密码错误！",})
              else:
                  return render(request, 'online/login.html', {'error_message': "用户名或密码错误！",})

          else:
              return render(request, 'online/login.html', {'error_message': "用户名或密码错误！",})
              
 <h2>二：在登录后需要用户在线状态校验的视图函数前加</h2>
   blogs/views.py
 
    # 主页
    @login_required
    def index(request, username):
        # print username
        userinfo = MyUser.objects.filter(username=username)
        all_objects = BlogsManage.objects.filter(author=username)
        objects, page_range = my_pagination(request, all_objects)
        return render_to_response('blogs/index.html', {'userinfo': userinfo, 'objects': objects, 'page_range': page_range})
 
 </body>
 </html>
