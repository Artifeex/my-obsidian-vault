A **sticky session**, also known as **session affinity**, is a technique used in load balancing to ensure that a user's requests are consistently directed to the same backend server during a session. This can be necessary when session-specific data, such as login information or a shopping cart, is stored on a single server and isn't shared across others.

### How Sticky Sessions Work

1. **Initial Request**: The load balancer routes the user's first request to one of the available backend servers.
2. **Session Persistence**: The load balancer creates a mapping between the user's session and the selected server, often using:
    - A **session cookie** (e.g., inserted by the application or load balancer).
    - The user's **IP address** (though this is less common and can be unreliable, especially with NAT or proxies).
3. **Subsequent Requests**: The load balancer uses the mapping to route all subsequent requests from the same user to the same backend server.

### Use Cases

- **Stateful Applications**: Applications that store session-related data on the server and do not share it across servers.
- **E-commerce**: To ensure a consistent shopping experience where user-specific cart details are on one server.
- **Authentication**: When user authentication state is maintained on a specific server.

### Implementation

Sticky sessions can be configured on most load balancers, such as:

- **NGINX**: Using `ip_hash` or by leveraging cookies.
- **HAProxy**: With the `stick-table` directive or session cookies.
- **AWS Elastic Load Balancer (ELB)**: Enabling session stickiness via cookies.

### Advantages

- Simplifies server-side session management for applications that are not fully stateless.
- Ensures a seamless user experience.

### Disadvantages

- **Scalability**: Can cause uneven load distribution because some servers may become overloaded while others remain underutilized.
- **Fault Tolerance**: If the assigned server fails, session data is lost unless mechanisms like session replication are in place.
- **Performance**: May slightly increase response times if the load balancer needs to perform additional checks for session affinity.

Sticky sessions are ideal for legacy or stateful applications, but modern applications often use distributed session storage or stateless designs to avoid reliance on sticky sessions.