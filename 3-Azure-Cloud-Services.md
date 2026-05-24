# Azure Cloud & Services

## Question 51: App Service vs AKS vs Functions

**Answer:**

| Aspect | App Service | AKS | Functions |
|--------|------------|-----|-----------|
| **Model** | Fully managed PaaS | Kubernetes cluster | Serverless |
| **Scaling** | Auto-scale instances | Pod-based | Automatic (event-driven) |
| **Cost** | Per app instance | Per VM/node | Pay per execution |
| **Complexity** | Low | High | Very low |
| **Cold start** | Seconds | Seconds | Milliseconds-seconds |
| **Best for** | Web apps, APIs | Microservices, complex apps | Event-driven, sporadic workloads |
| **DevOps effort** | Minimal | Extensive | Minimal |

**App Service - Traditional Web Apps:**
```csharp
// Simple web API
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
var app = builder.Build();
app.MapControllers();
app.Run();

// Deploy: Right-click → Publish → App Service
```

**AKS - Microservices:**
```yaml
# Deployment manifest
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: myregistry.azurecr.io/order-service:latest
        ports:
        - containerPort: 5000
```

**Functions - Event-Driven:**
```csharp
[FunctionName("ProcessOrder")]
public static async Task Run(
    [QueueTrigger("order-queue")] Order order,
    [ServiceBus("order-notifications", Connection = "ServiceBusConnection")] IAsyncCollector<OrderNotification> notifications,
    ILogger log) {
    
    log.LogInformation($"Processing order {order.Id}");
    
    // Process
    var result = await ProcessOrderAsync(order);
    
    // Notify
    await notifications.AddAsync(new OrderNotification {
        OrderId = order.Id,
        Status = "Processed"
    });
}
```

---

## Question 52: Consumption vs Premium Functions

**Answer:**

| Feature | Consumption | Premium |
|---------|-------------|---------|
| **Cost** | Per execution + GB-s | Fixed price/month |
| **Timeout** | 5 minutes | 10 minutes |
| **Memory** | 128-1536 MB | 128-3072 MB |
| **VNet support** | No | Yes |
| **Cold start** | Significant | Minimal |
| **Concurrency** | Limited | Higher |
| **Always On** | No | Yes |

**Use Consumption When:**
- Unpredictable, sporadic usage
- Short-running functions (<5 min)
- Cost-conscious
- Event-driven workloads

**Use Premium When:**
- Predictable, continuous usage
- Need VNet access
- Minimize cold starts
- Long-running functions
- Higher throughput required

---

## Question 53: Managed Identity benefits

**Answer:**

**No Secrets in Code:**
```csharp
// Before - secrets in code/config
var client = new SecretClient(
    new Uri("https://myvault.vault.azure.net/"),
    new ClientSecretCredential(tenantId, clientId, clientSecret)
);

// After - Managed Identity (no secrets!)
var client = new SecretClient(
    new Uri("https://myvault.vault.azure.net/"),
    new DefaultAzureCredential() // Uses system identity
);
```

**Benefits:**
- No secrets to manage
- Automatic token rotation
- Role-based access control
- Audit trail
- Secure by default

**Setup:**
```bash
# Enable managed identity on App Service
az webapp identity assign --name myapp --resource-group mygroup

# Grant permissions
az role assignment create \
  --assignee-object-id <object-id> \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<vault>"
```

---

## Question 54: Key Vault secret rotation

**Answer:**

**Automatic Rotation:**
```csharp
// Event Grid triggers rotation function
[FunctionName("RotateSecret")]
public static async Task Run(
    [EventGridTrigger] EventGridEvent eventGridEvent,
    ILogger log) {
    
    var secretName = eventGridEvent.Subject;
    
    // Generate new secret
    var newSecret = GenerateNewSecret();
    
    // Update Key Vault
    await _secretClient.SetSecretAsync(secretName, newSecret);
    
    // Update dependent resources
    await UpdateAppServiceSettingAsync(secretName, newSecret);
    await RestartAppServiceAsync();
}
```

**Application-Level Rotation:**
```csharp
// Periodically refresh secrets
public class SecretRotationBackgroundService : BackgroundService {
    protected override async Task ExecuteAsync(CancellationToken cancellationToken) {
        while (!cancellationToken.IsCancellationRequested) {
            // Refresh secrets every 30 days
            var secret = await _secretClient.GetSecretAsync("DatabasePassword");
            
            // Store in memory cache with shorter TTL
            _cache.Set("dbPassword", secret.Value.Value, TimeSpan.FromHours(1));
            
            await Task.Delay(TimeSpan.FromDays(30), cancellationToken);
        }
    }
}
```

---

## Question 55: Blob storage tiers

**Answer:**

| Tier | Access | Cost/month | Use Case |
|------|--------|-----------|----------|
| **Hot** | Frequent | Highest storage | Active data |
| **Cool** | Infrequent | Medium | 30+ days |
| **Archive** | Rarely | Lowest | Long-term backup |

**Lifecycle Management:**
```json
{
  "rules": [
    {
      "name": "auto-tier",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 90 },
            "delete": { "daysAfterModificationGreaterThan": 365 }
          }
        },
        "filters": { "blobTypes": ["blockBlob"] }
      }
    }
  ]
}
```

---

## Question 56: SAS vs RBAC

**Answer:**

**SAS (Shared Access Signature) - Time-limited token:**
```csharp
var sasUri = _blobClient.GenerateSasUri(BlobSasPermissions.Read, DateTimeOffset.UtcNow.AddHours(1));
// sasUri = "https://...?sv=2021-06-08&se=...&sig=..."

// Give to client - expires after 1 hour
return Ok(new { sasUri });
```

**RBAC (Role-Based Access Control) - Identity-based:**
```csharp
// User with "Storage Blob Data Reader" role
var client = new BlobClient(
    new Uri("https://storage.blob.core.windows.net/container/blob"),
    new DefaultAzureCredential()
);
var blob = await client.DownloadAsync();
```

**Comparison:**

| Aspect | SAS | RBAC |
|--------|-----|------|
| **Time limit** | Yes | No (role-based) |
| **Granularity** | Fine (container/blob) | Coarse (role-level) |
| **Revocation** | Time-based | Immediate |
| **Auditing** | Limited | Full |
| **Use case** | Client delegation | Service-to-service |

---

## Question 57: Service Bus queues vs topics

**Answer:**

**Queue - Point-to-Point:**
```csharp
// Single consumer per message
var sender = _client.CreateSender("order-queue");
await sender.SendMessageAsync(new ServiceBusMessage("process-order"));

var processor = _client.CreateProcessor("order-queue");
processor.ProcessMessageAsync += async args => {
    // Only one consumer processes
    await ProcessOrderAsync(args.Message);
};
```

**Topic - Pub/Sub:**
```csharp
// Multiple subscribers get same message
var sender = _client.CreateSender("order-events");
await sender.SendMessageAsync(new ServiceBusMessage("order-created"));

// Subscriber 1
var processor1 = _client.CreateProcessor("order-events", "notification-subscription");
processor1.ProcessMessageAsync += async args => {
    // Send notification
    await SendNotificationAsync(args.Message);
};

// Subscriber 2
var processor2 = _client.CreateProcessor("order-events", "inventory-subscription");
processor2.ProcessMessageAsync += async args => {
    // Update inventory
    await UpdateInventoryAsync(args.Message);
};
```

**Comparison:**

| Feature | Queue | Topic |
|---------|-------|-------|
| **Pattern** | Point-to-point | Publish-subscribe |
| **Consumers** | One | Many |
| **Message retention** | Until consumed | Per subscription |
| **Filtering** | By property | By subscription |

---

## Question 58: Event Grid vs Event Hub

**Answer:**

**Event Grid - Event Routing:**
```csharp
// Automatic routing based on event type
[EventGridTrigger]
public static void OnBlobCreated(EventGridEvent @event) {
    var storageEvent = JsonConvert.DeserializeObject<StorageBlobCreatedEventData>(@event.Data.ToString());
    // Process blob
}

// Event subscriptions defined in portal
// Event Grid routes to appropriate handlers
```

**Event Hub - High-throughput Streaming:**
```csharp
// Continuous data stream
var producer = new EventHubProducerClient("connection-string", "my-hub");
var batch = await producer.CreateBatchAsync();

for (int i = 0; i < 1000; i++) {
    batch.TryAdd(new EventData(Encoding.UTF8.GetBytes($"Event {i}")));
}

await producer.SendAsync(batch);

// Consumer
var consumer = new EventHubConsumerClient(EventHubConsumerClient.DefaultConsumerGroupName, "connection-string", "my-hub");

await foreach (PartitionEvent partitionEvent in consumer.ReadEventsAsync()) {
    // Process streaming data
}
```

**Comparison:**

| Aspect | Event Grid | Event Hub |
|--------|-----------|-----------|
| **Throughput** | Low-medium | High |
| **Pattern** | Event routing | Stream processing |
| **Latency** | ~50ms | Real-time |
| **Retention** | Minutes | Hours/days |
| **Use case** | Event reactions | Telemetry, IoT |

---

## Question 59: Cosmos DB partition key design

**Answer:**

**Choosing Good Partition Keys:**

```csharp
// Good: High cardinality, evenly distributed
public class Order {
    public string Id { get; set; } // Unique per order
    public string CustomerId { get; set; } // Partition key
    public DateTime CreatedAt { get; set; }
}

// Usage: Queries filtered by CustomerId go to single partition
var orders = container
    .GetItemLinqQueryable<Order>()
    .Where(o => o.CustomerId == "cust123")
    .ToList();
```

**Bad Partition Key:**
```csharp
// Bad: Low cardinality
public string IsActive { get; set; } // Only true/false - uneven distribution
public string Country { get; set; } // Only ~200 values - hot partitions
public bool IsPremium { get; set; } // Binary - 50/50 split creates hot partition
```

**Hierarchical Partition Key (C# 10):**
```csharp
public class Document {
    [JsonPropertyName("id")]
    public string Id { get; set; }
    
    [JsonPropertyName("customerId")]
    public string CustomerId { get; set; }
    
    [JsonPropertyName("orderId")]
    public string OrderId { get; set; }
    
    [JsonPropertyName("timestamp")]
    public DateTime Timestamp { get; set; }
}

// Composite partition key: /customerId/orderId
// Enables multi-level filtering
container
    .GetItemLinqQueryable<Document>()
    .Where(d => d.CustomerId == "cust1" && d.OrderId == "order1")
    .ToList();
```

---

## Question 60: RU/s optimization

**Answer:**

**Understanding RUs (Request Units):**
- 1 RU = resources to read 1 KB item
- Write = ~5 RUs
- Large items = more RUs

**Reduce RU Consumption:**
```csharp
// 1. Use smaller documents
public class Order {
    public string Id { get; set; }
    public string CustomerId { get; set; }
    public decimal Total { get; set; }
    // Don't store nested large arrays
}

// 2. Use single-partition queries (most efficient)
// Good: Partition key in WHERE clause
var orders = container
    .GetItemLinqQueryable<Order>()
    .Where(o => o.CustomerId == "cust1") // Partition key
    .ToList(); // ~1 RU

// Bad: Cross-partition query
var allOrders = container
    .GetItemLinqQueryable<Order>()
    .Where(o => o.Total > 100) // Not partition key
    .ToList(); // ~1000 RU for same data

// 3. Use point reads (most efficient)
var order = await container.ReadItemAsync<Order>("order1", new PartitionKey("cust1")); // ~1 RU

// 4. Batch operations
var batch = new TransactionalBatch(container)
    .CreateItem(new Order { ... })
    .UpdateItem("order1", updatedOrder)
    .DeleteItem("order2");

await batch.ExecuteAsync(); // Single transactional batch
```

---

## Question 61: Consistency levels in Cosmos DB

**Answer:**

| Level | Guarantees | Latency | Use Case |
|-------|-----------|---------|----------|
| **Strong** | Latest | High | Financial, critical |
| **Bounded staleness** | Within limits | Medium | Analytics, reporting |
| **Session** | Within session | Low | Web apps |
| **Consistent prefix** | Order preserved | Very low | Event streams |
| **Eventual** | Eventually consistent | Lowest | Social media |

**Strong Consistency:**
```csharp
// Always reads latest
var order = await container.ReadItemAsync<Order>("order1", new PartitionKey("cust1"));
// Guaranteed to have all previous writes
```

**Session Consistency (Default):**
```csharp
// Reads see your writes
public class OrderService {
    public async Task<Order> CreateAndGetAsync(Order order) {
        // Write
        await container.CreateItemAsync(order);
        
        // Read sees write (same session)
        var saved = await container.ReadItemAsync<Order>(order.Id, new PartitionKey(order.CustomerId));
        return saved;
    }
}
```

**Eventual Consistency:**
```csharp
// Fastest, but temporary inconsistency possible
var item1 = await container.ReadItemAsync<Order>(id, key); // Might see old version
var item2 = await Task.Delay(100).ContinueWith(async _ =>
    await container.ReadItemAsync<Order>(id, key) // Likely sees new version
);
```

---

## Question 62: Hot partition troubleshooting

**Answer:**

**Identifying Hot Partitions:**
```csharp
// Monitor partition metrics
var metrics = await container.GetDiagnostics();
// Look for high RU consumption from single partition

// Check Metrics blade in Portal
// Azure Cosmos DB → Insights → Normalized RU Consumption
```

**Causes:**
- Poor partition key (low cardinality)
- Skewed data distribution
- Hot customer/tenant
- Burst traffic patterns

**Solutions:**

**1. Change Partition Key:**
```csharp
// Bad: Low cardinality
public string Region { get; set; } // Only few regions

// Good: High cardinality
public string CustomerId { get; set; } // Unique per customer
```

**2. Migrate Data:**
```csharp
// Recreate collection with new partition key
var options = new ContainerProperties {
    Id = "orders-v2",
    PartitionKeyPath = "/customerId"
};

var container = await database.CreateContainerAsync(options, 10000);

// Migrate data
foreach (var item in oldContainer.GetItemLinqQueryable<Order>()) {
    await container.CreateItemAsync(item);
}
```

**3. Synthetic Partition Key:**
```csharp
public class Order {
    public string Id { get; set; }
    public string CustomerId { get; set; }
    public string SyntheticKey { get; set; } = Guid.NewGuid().ToString(); // Add random partition
    
    // Use SyntheticKey as partition key for distribution
}
```

---

## Question 63: Azure Monitor vs App Insights

**Answer:**

**Azure Monitor - Infrastructure:**
```csharp
// Monitor VMs, networks, databases
var metric = new Metric {
    Name = "cpu_percentage",
    Value = 75,
    Timestamp = DateTime.UtcNow,
    ResourceId = "/subscriptions/.../providers/Microsoft.Compute/virtualMachines/myvm"
};

// Set alerts on infrastructure metrics
// Alert when CPU > 80%
```

**Application Insights - Application:**
```csharp
public class TelemetryController : ControllerBase {
    private readonly TelemetryClient _telemetryClient;
    
    [HttpGet("{id}")]
    public IActionResult Get(int id) {
        _telemetryClient.TrackEvent("UserFetch", new Dictionary<string, string> {
            { "userId", id.ToString() }
        });
        
        return Ok();
    }
}

// Monitors: Requests, dependencies, exceptions, custom events
```

**Comparison:**

| Aspect | Monitor | App Insights |
|--------|---------|--------------|
| **Scope** | Infrastructure | Application |
| **Metrics** | CPU, memory, disk | Requests, performance |
| **Logs** | Platform logs | App logs |
| **Analytics** | KQL queries | KQL + AI |
| **Integration** | Azure services | Code instrumentation |

---

## Question 64: Availability zones vs sets

**Answer:**

**Availability Sets - Same Region, Different Hardware:**
```bash
az vm create \
  --name myvm \
  --availability-set myavset \
  # Spreads across different hardware in same region
  # Protects against: hardware failure, maintenance
  # Limitation: Single region
```

**Availability Zones - Different Locations:**
```bash
az vm create \
  --name myvm \
  --zone 1 \
  # Place in separate zone (1, 2, or 3)
  # Each zone = different data center
  # Protects against: data center failure
  # Better redundancy than availability sets
```

**Comparison:**

| Feature | Avail Sets | Avail Zones |
|---------|-----------|------------|
| **Scope** | Same region | Same region, different locations |
| **Failure protection** | Hardware | Data center |
| **SLA** | 99.95% | 99.99% |
| **Latency** | Minimal | Low |
| **Cost** | Same | Possible egress charges |

**Recommendation:**
Use both - Availability Zones for primary redundancy, Availability Sets for additional resilience within zone.

---

## Question 65: Private endpoint use cases

**Answer:**

**Problem: Public Endpoint Exposes Service:**
```csharp
// Public endpoint accessible from internet
// https://myaccount.blob.core.windows.net/
// Vulnerable to attacks
```

**Solution: Private Endpoint:**
```bash
# Create private endpoint in VNet
az network private-endpoint create \
  --name myendpoint \
  --resource-group mygroup \
  --vnet-name myvnet \
  --subnet mysubnet \
  --private-connection-resource-id "/subscriptions/.../providers/Microsoft.Storage/storageAccounts/myaccount" \
  --group-id "blob"

# Now accessible only from within VNet
# 10.0.0.0/24 → 10.0.1.5 (private IP)
```

**Benefits:**
- No public IP exposure
- Traffic stays in Azure backbone
- Lower latency
- Better security posture

---

## Question 66: VNet integration

**Answer:**

**App Service VNet Integration:**
```bash
az webapp vnet-integration add \
  --name myapp \
  --resource-group mygroup \
  --vnet vnetname \
  --subnet subnetname

# App can now access:
# - Private databases
# - Internal services
# - On-premises resources (via VPN)
```

**Benefits:**
- Access private SQL servers
- Connect to on-premises
- Network isolation
- Compliance requirements

---

## Question 67: NSG rules

**Answer:**

**Network Security Group - Firewall Rules:**

```bash
# Allow HTTP from internet
az network nsg rule create \
  --nsg-name mynsg \
  --name "allow-http" \
  --priority 1000 \
  --direction Inbound \
  --source-address-prefixes "0.0.0.0/0" \
  --destination-port-ranges 80 \
  --access Allow

# Allow SSH from specific IP
az network nsg rule create \
  --nsg-name mynsg \
  --name "allow-ssh-admin" \
  --priority 100 \
  --source-address-prefixes "203.0.113.0/32" \
  --destination-port-ranges 22 \
  --access Allow

# Deny all else
# (Default implicit deny)
```

---

## Question 68: Azure Front Door vs Application Gateway

**Answer:**

| Aspect | Front Door | App Gateway |
|--------|-----------|------------|
| **Scope** | Global (Layer 7) | Regional (Layer 7) |
| **Load balancing** | Global routing | Regional routing |
| **Failover** | Automatic global | Within region |
| **Use case** | Multi-region apps | Single region, complex rules |

---

## Question 69: Disaster recovery strategy

**Answer:**

**RTO (Recovery Time Objective) & RPO (Recovery Point Objective):**

```
Original → Failure → Detection → Recovery → Restored
    |              |             |        |
    <--RTO--------><--Detection--|
    <--RPO--------><-- Data loss|
```

**Strategies:**

**1. Backup & Restore (Cheapest)**
- RTO: Hours/days
- RPO: Days
- Use case: Non-critical systems

**2. Pilot Light**
- Minimal standby infrastructure
- RTO: 15-30 minutes
- RPO: Minutes

**3. Warm Standby**
- Secondary region with reduced capacity
- RTO: 5-15 minutes
- RPO: Real-time replication

**4. Hot Standby**
- Full duplicate in secondary region
- RTO: <1 minute
- RPO: ~0 (continuous replication)
- Most expensive

**Implementation:**
```csharp
// Azure Site Recovery
// Replicates VMs to secondary region
// Automatic failover on primary failure

// Cosmos DB
// Multi-region writes for disaster recovery
var cosmosClient = new CosmosClient(connectionString, new CosmosClientOptions {
    ApplicationRegion = "West US" // Preferred region
});

// Database automatically replicates to other regions
```

---

## Question 70: Blue-green deployment in Azure

**Answer:**

**Strategy: Two identical environments, switch traffic:**

```
Blue Environment (Current)  Green Environment (New)
├─ app-blue-slot           ├─ app-green-slot
├─ Database (shared)        ├─ Database (shared)
└─ Traffic router ←────────────→ Directs to one
```

**Implementation:**

```bash
# Deploy to staging slot
az webapp deployment slot create \
  --name myapp \
  --resource-group mygroup \
  --slot staging

# Deploy new version to staging
az webapp up \
  --name myapp \
  --slot staging \
  --plan myplan

# Test staging deployment
# https://myapp-staging.azurewebsites.net

# Swap slots (blue ← green)
az webapp deployment slot swap \
  --name myapp \
  --resource-group mygroup

# Production now runs new version
# If issues, swap back immediately
```

**PowerShell:**
```powershell
# Deploy to slot
Publish-AzWebapp -ResourceGroupName mygroup -Name myapp -ArchivePath package.zip -Slot staging

# Verify staging
# https://myapp-staging.azurewebsites.net

# Swap to production
Switch-AzWebAppSlot -ResourceGroupName mygroup -Name myapp -SourceSlot staging
```

---

## Question 71: Cost optimization practices

**Answer:**

**1. Right-size resources:**
```bash
# Review underutilized VMs
az vm show --name myvm --resource-group mygroup # Check CPU utilization
# If <10%, downsize

# Example: Standard_D4s_v3 → Standard_D2s_v3 (50% cost saving)
```

**2. Use reserved instances:**
```bash
# Save 40% with 3-year commitment
az reservations purchase \
  --scope shared \
  --sku Standard_D2s_v3 \
  --term P3Y
```

**3. Turn off non-prod resources:**
```bash
# Auto-shutdown dev resources after hours
az vm auto-shutdown \
  --name devvm \
  --time 18:00 \
  --location eastus
```

**4. Use Azure Hybrid Benefit:**
```bash
# Use existing SQL/Windows licenses in Azure
# Save 40-55% on compute
```

**5. Monitor costs:**
```bash
# Set budget alerts
az billing subscription \
  --set-budget 1000 --alert-threshold 80
```

---

## Question 72: ARM vs Bicep vs Terraform

**Answer:**

**ARM (Azure Resource Manager) - Native JSON:**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2021-03-01",
      "name": "myvm",
      "location": "[resourceGroup().location]",
      "properties": {}
    }
  ]
}
```

**Bicep - Simplified ARM:**
```bicep
param location string = resourceGroup().location
param vmName string = 'myvm'

resource vm 'Microsoft.Compute/virtualMachines@2021-03-01' = {
  name: vmName
  location: location
  properties: {}
}
```

**Terraform - Multi-cloud:**
```hcl
terraform {
  required_providers {
    azurerm = {
      source = "hashicorp/azurerm"
    }
  }
}

resource "azurerm_virtual_machine" "vm" {
  name                  = "myvm"
  location              = azurerm_resource_group.rg.location
  resource_group_name   = azurerm_resource_group.rg.name
}
```

**Comparison:**

| Aspect | ARM | Bicep | Terraform |
|--------|-----|-------|-----------|
| **Learning** | Steep | Medium | Medium |
| **Azure-specific** | Yes | Yes | Multi-cloud |
| **Readability** | Poor | Good | Good |
| **Community** | Large | Growing | Very large |
| **State management** | None | None | Needed |

---

## Question 73: Azure DevOps pipelines

**Answer:**

**CI/CD Pipeline:**
```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'

stages:
- stage: Build
  jobs:
  - job: BuildApplication
    steps:
    - task: UseDotNet@2
      inputs:
        version: '6.0.x'
    
    - task: DotNetCoreCLI@2
      inputs:
        command: 'build'
        arguments: '--configuration $(buildConfiguration)'
    
    - task: DotNetCoreCLI@2
      inputs:
        command: 'test'
    
    - task: DotNetCoreCLI@2
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
    
    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'

- stage: Deploy_Staging
  dependsOn: Build
  jobs:
  - deployment: DeployApp
    environment: staging
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Azure Subscription'
              appName: 'myapp-staging'
              package: '$(Pipeline.Workspace)/drop'

- stage: Deploy_Production
  dependsOn: Deploy_Staging
  jobs:
  - deployment: DeployApp
    environment: production
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              appName: 'myapp-prod'
              package: '$(Pipeline.Workspace)/drop'
```

---

## Question 74: Zero trust architecture

**Answer:**

**Never trust, always verify:**

1. **Verify Identity** - Multi-factor authentication
2. **Verify Device** - Check device compliance
3. **Verify Network** - Encrypt all traffic
4. **Verify Access** - Least privilege (just-in-time)

**Implementation:**
```csharp
// Azure AD conditional access
[Authorize(Policy = "ZeroTrust")]
public class AdminController {
    // Requires:
    // - MFA
    // - Compliant device
    // - Approved IP range
}

// Just-in-time access
az role assignment create \
  --assignee user@company.com \
  --role "Virtual Machine Contributor" \
  --scope "/subscriptions/.../resourceGroups/rg1" \
  --duration "PT1H" # 1 hour access
```

**Key Principles:**
- Assume breach
- Verify explicitly
- Least privilege access
- Encrypt everything
- Monitor & log all access

---

## Question 75: Design a secure enterprise Azure platform

**Answer:**

**Architecture:**
```
Internet → Azure Front Door (WAF)
           ↓
        API Gateway (Auth)
           ↓
    Managed Identity ← Azure AD
           ↓
    App Service (VNet integrated)
           ↓
    Private Endpoint
           ↓
    Azure SQL (encrypted, private)
           ↓
    Key Vault (secrets rotation)
           ↓
    Storage Account (private, encrypted)
```

**Implementation:**

```bash
# Network isolation
az network vnet create --name corenet --resource-group mygroup --address-prefix 10.0.0.0/16
az network subnet create --vnet-name corenet --name appsubnet --address-prefix 10.0.1.0/24
az network subnet create --vnet-name corenet --name dbsubnet --address-prefix 10.0.2.0/24

# NSG - Allow only required traffic
az network nsg create --name appsecurity --resource-group mygroup
az network nsg rule create --nsg-name appsecurity --priority 100 \
  --source-address-prefixes "10.0.0.0/16" \
  --destination-port-ranges 443 --access Allow

# Private endpoints for data
az network private-endpoint create --name sqlendpoint \
  --vnet-name corenet --subnet dbsubnet \
  --private-connection-resource-id "/subscriptions/.../sqlServers/mysql"

# Managed Identity for authentication
az app service identity assign --name myapp --resource-group mygroup
az role assignment create --assignee-object-id <msi-id> \
  --role "SQL DB Contributor" --scope "/subscriptions/.../sqlServers/mysql"

# Encryption everywhere
az storage account update --name mysa --resource-group mygroup \
  --encryption-services blob --encryption-key-type-for-table Account

# Monitoring & compliance
az monitor diagnostic-settings create --name audit \
  --resource-group mygroup --resource myresource \
  --logs '[{"category":"AuditLogs","enabled":true}]'

# Azure Policy for compliance
az policy assignment create --name "require-encryption" \
  --policy-definition-id "/subscriptions/.../providers/Microsoft.Authorization/policyDefinitions/..."
```

**Security Features:**
- VNet isolation
- Private endpoints (no public IPs)
- Managed identities (no secrets)
- TLS encryption in transit
- Transparent data encryption (TDE) at rest
- Azure AD integration
- Multi-factor authentication
- Role-based access control (RBAC)
- Audit logging
- Network security groups (firewalls)
- Web application firewall (WAF)
- Continuous compliance monitoring
- Secrets rotation in Key Vault

