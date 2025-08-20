# online-food-ordering-website
# online-food-ordering-website
 
django-admin startproject food_ordering
cd food_ordering
python manage.py startapp orders

food ordering website
from django.db import models
from django.contrib.auth.models import User

# Food Menu
class FoodItem(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=6, decimal_places=2)

    def __str__(self):
        return self.name

# Customer Order
class Order(models.Model):
    customer = models.ForeignKey(User, on_delete=models.CASCADE)
    items = models.ManyToManyField(FoodItem)
    total_price = models.DecimalField(max_digits=8, decimal_places=2)
    status = models.CharField(
        max_length=20,
        choices=[("Pending", "Pending"), ("Accepted", "Accepted"), ("Delivered", "Delivered")],
        default="Pending"
    )
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"Order {self.id} by {self.customer.username}"
from django.shortcuts import render, redirect
from .models import FoodItem, Order
from django.contrib.auth.decorators import login_required

# Show Menu
@login_required
def menu(request):
    foods = FoodItem.objects.all()
    return render(request, 'menu.html', {'foods': foods})

# Place Order
@login_required
def place_order(request):
    if request.method == "POST":
        food_ids = request.POST.getlist("foods")
        items = FoodItem.objects.filter(id__in=food_ids)
        total = sum([item.price for item in items])
        order = Order.objects.create(customer=request.user, total_price=total)
        order.items.set(items)
        return redirect("order_history")
    foods = FoodItem.objects.all()
    return render(request, "place_order.html", {"foods": foods})

# Show Order History
@login_required
def order_history(request):
    orders = Order.objects.filter(customer=request.user)
    return render(request, "order_history.html", {"orders": orders})

# Reception/Admin → See All Orders
@login_required
def all_orders(request):
    orders = Order.objects.all()
    return render(request, "all_orders.html", {"orders": orders})



<h2>Menu</h2>
<form method="POST" action="{% url 'place_order' %}">
  {% csrf_token %}
  {% for food in foods %}
    <input type="checkbox" name="foods" value="{{ food.id }}">
    {{ food.name }} - ₹{{ food.price }} <br>
  {% endfor %}
  <button type="submit">Place Order</button>
</form>

