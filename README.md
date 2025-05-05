# Monthly_bill_generator
This Python script generates a monthly bill by calculating prorated charges for items active during a specified month. It processes input data (item details, start/stop dates), calculates the active days, groups similar items, and returns a detailed breakdown of line items and total revenue
# 📘 Monthly Bill Generator – Python Assessment

# ✅ Step 1: Import necessary modules
# These are the tools we need to handle dates and group items together easily
from datetime import datetime, timedelta
from collections import defaultdict
from pprint import pprint  # This is for displaying the final result in a more readable format

# ✅ Step 2: Define the item list
# This is the data for each item. Each item has start and stop dates, quantity, rate, etc.
item_list = [
    # Same item list as provided in your original example
    {"idx": 1, "item_code": "Executive Desk (4*2)", "sales_description": "Dedicated Executive Desk", "qty": 10, "rate": "1000", "amount": "10000", "start_date": "2023-11-01", "stop_date": "2024-10-17"},
    {"idx": 2, "item_code": "Executive Desk (4*2)", "qty": "10", "rate": "1080", "amount": "10800", "start_date": "2024-10-18", "stop_date": "2025-10-31"},
    {"idx": 3, "item_code": "Executive Desk (4*2)", "qty": 15, "rate": "1080", "amount": "16200", "start_date": "2024-11-01", "stop_date": "2025-10-31"},
    # ... more items here (same as your original list)
]

# ✅ Step 3: Define the main function to generate the monthly bill
def generate_monthly_bill(item_list: list, target_month: str) -> dict:
    """
    This function calculates the monthly bill based on the given item list and target month.
    
    - item_list: List of items with start and stop dates, rate, quantity, etc.
    - target_month: The month for which the bill is being generated, in the format "YYYY-MM".
    """
    
    # Extract the year and month from the target_month string (e.g., "2024-11" -> year=2024, month=11)
    year, month = map(int, target_month.split('-'))
    
    # Define the start and end date for the given month
    month_start = datetime(year, month, 1)
    month_end = datetime(year + (month // 12), (month % 12) + 1, 1) - timedelta(days=1)
    
    # This dictionary will group similar items by item code, rate, and billing period
    grouped = defaultdict(lambda: {"qty": 0, "amount": 0.0})
    
    # This will hold the total revenue for the month
    total_revenue = 0.0

    # Loop through each item in the item list
    for item in item_list:
        # Convert the start and stop dates from string to datetime objects for comparison
        item_start = datetime.strptime(item['start_date'], "%Y-%m-%d")
        item_end = datetime.strptime(item['stop_date'], "%Y-%m-%d")
        
        # Find the overlap between the item's active period and the target month
        active_start = max(month_start, item_start)
        active_end = min(month_end, item_end)
        
        # If there’s no overlap, skip this item
        if active_start > active_end:
            continue
        
        # Calculate how many days the item was active during the target month
        active_days = (active_end - active_start).days + 1
        total_days = (month_end - month_start).days + 1
        
        # Find the fraction of the month the item was active
        active_ratio = active_days / total_days
        
        # Convert rate and quantity to numbers (in case they were given as strings)
        rate = float(item['rate'])
        qty = int(item['qty'])
        
        # Calculate the prorated amount for the item during the active period
        amount = round(rate * qty * active_ratio, 2)
        
        # Format the billing period as a string (e.g., "2024-11-01 to 2024-11-30")
        billing_period = f"{active_start.date()} to {active_end.date()}"
        
        # Group the item by its code, rate, and the active billing period
        key = (item['item_code'], rate, billing_period)
        
        # Accumulate the quantity and amount for this item
        grouped[key]['qty'] += qty
        grouped[key]['amount'] += amount
    
    # Prepare the final list of line items to be displayed in the bill
    line_items = []
    for (item_code, rate, billing_period), data in grouped.items():
        line_items.append({
            "item_code": item_code,
            "rate": rate,
            "qty": data['qty'],
            "amount": round(data['amount'], 2),
            "billing_period": billing_period
        })
        
        # Add the item amount to the total revenue
        total_revenue += data['amount']

    # Return the final billing summary with all line items and total revenue
    return {
        "line_items": line_items,
        "total_revenue": round(total_revenue, 2)
    }

# ✅ Step 4: Run the function for the month of November 2024 and print the result
result = generate_monthly_bill(item_list, "2024-11")
pprint(result)  # This will print the result in a readable format
