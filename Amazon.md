<b>Online Shopping System</b><br>

Build an online shopping system that stores an inventory of products under different categories, allow customers to search through them, and make purchases.
The system handle the expanding load on the website and prevent it from crashing down, particularly on busy days, such as the Black Friday Sale.

<b>Functional Requirements</b><br>
Sellers should be able to add, delete and modify products they want to sell.<br>
The website should include a catalog of products.<br>
Buyers can search products by name, keyword or category.<br>
Buyers can add, delete or update items in a cart.<br>
Buyers can purchase items in the cart and make payments.<br>
Buyers can view their previous orders.<br>
Buyers can review and rate purchased products.<br>

<b>Non Functional Requirements</b><br>
High availability<br>
High consistency<br>
Low latency<br>
Maintaining all three non-functional requirements at all times is difficult, especially with the high traffic that the website will need to handle. However, different components of the system architecture can have different priorities.<br>
Payment service and inventory management will need to be highly consistent. Availability is of low priority in this case. <br>
Search service, on the other hand, should be highly available and have low latency, even at the cost of consistency. In other words, it’s acceptable if the search service is eventually consistent.<br>

Amazon System Architecture
If you are a customer of Amazon, you can either be selling products through your own isolated store on Amazon, or you can be buying products available from different sellers.



reference: https://medium.com/double-pointer/system-design-interview-amazon-flipkart-ebay-or-similar-e-commerce-applications-35a0bc764421
