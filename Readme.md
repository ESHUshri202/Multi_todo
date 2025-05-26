for performance system 
models.py 
from django.db import models
from django.contrib.auth.models import User

class Employee(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    department = models.CharField(max_length=100)
    role = models.CharField(max_length=100)

    def __str__(self):
        return self.user.get_full_name()

class Goal(models.Model):
    employee = models.ForeignKey(Employee, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    description = models.TextField()
    target_date = models.DateField()
    completed = models.BooleanField(default=False)

    def __str__(self):
        return self.title

class PerformanceReview(models.Model):
    employee = models.ForeignKey(Employee, on_delete=models.CASCADE)
    reviewer = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
    review_date = models.DateField(auto_now_add=True)
    rating = models.PositiveIntegerField()  # scale of 1 to 5
    comments = models.TextField()

class Feedback(models.Model):
    sender = models.ForeignKey(User, related_name='sent_feedback', on_delete=models.CASCADE)
    receiver = models.ForeignKey(User, related_name='received_feedback', on_delete=models.CASCADE)
    message = models.TextField()
    date_sent = models.DateTimeField(auto_now_add=True)


admin.py
from django.contrib import admin
from .models import Employee, Goal, PerformanceReview, Feedback

admin.site.register(Employee)
admin.site.register(Goal)
admin.site.register(PerformanceReview)
admin.site.register(Feedback)



views.py and forms.py

# forms.py
from django import forms
from .models import Goal, PerformanceReview

class GoalForm(forms.ModelForm):
    class Meta:
        model = Goal
        fields = ['title', 'description', 'target_date', 'completed']

class ReviewForm(forms.ModelForm):
    class Meta:
        model = PerformanceReview
        fields = ['rating', 'comments']


# views.py
from django.shortcuts import render, redirect
from .models import Goal, PerformanceReview, Employee
from .forms import GoalForm, ReviewForm
from django.contrib.auth.decorators import login_required

@login_required
def dashboard(request):
    employee = Employee.objects.get(user=request.user)
    goals = Goal.objects.filter(employee=employee)
    return render(request, 'performance/dashboard.html', {'goals': goals})

@login_required
def add_goal(request):
    if request.method == 'POST':
        form = GoalForm(request.POST)
        if form.is_valid():
            goal = form.save(commit=False)
            goal.employee = Employee.objects.get(user=request.user)
            goal.save()
            return redirect('dashboard')
    else:
        form = GoalForm()
    return render(request, 'performance/add_goal.html', {'form': form})



html 
<h1>Welcome, {{ user.get_full_name }}</h1>
<h2>Your Goals:</h2>
<ul>
  {% for goal in goals %}
    <li>{{ goal.title }} - {{ goal.target_date }} - {{ goal.completed }}</li>
  {% endfor %}
</ul>
<a href="{% url 'add_goal' %}">Add New Goal</a>



urls

# performance/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.dashboard, name='dashboard'),
    path('add-goal/', views.add_goal, name='add_goal'),
]

project url.py
path('performance/', include('performance.urls')),

