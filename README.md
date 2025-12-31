<p align="center">
  <img src="assets/metalblockchain-logo.png" alt="Metal Blockchain Logo" width="200" />
</p>

# Deploy Metal Validator Monitoring

Monitor your Metal blockchain validator node with Prometheus, Grafana, and node_exporter on Akash's decentralized cloud. Get real-time metrics, dashboards, and alerts for your validator deployment.

## Requirements

Before deploying, ensure you have:

- Metal validator node already deployed and running
- Validator IP address (from Akash Console → Deployment → "IP(s)" field)
- Akash wallet with AKT tokens (for monitoring deployment costs)
- Provider with `ip-lease: true` attribute (for direct IP access - recommended)
- 5-10 minutes for deployment and setup

## Cost

Monitoring deployment adds approximately:
- **Prometheus**: ~$2-5/month
- **Grafana**: ~$1-3/month
- **Node Exporter**: ~$0.50-1/month
- **Total**: ~$3.50-9/month additional cost

## Quick Deploy

1. Go to [Akash Console](https://console.akash.network)
2. Click "Deploy" → "Upload SDL"
3. Upload `deploy.yaml` from this repository
4. Update `METAL_NODE_URL` with your validator IP (see Step 2 below)
5. Deploy and wait 2-3 minutes

**GitHub Raw URL:**
```
https://raw.githubusercontent.com/VirgilBB/Metal-Grafana/main/deploy.yaml
```

## Deployment Steps

### Step 1: Get Your Validator IP

1. Go to **Akash Console** → Your Metal Validator Deployment
2. Look for the **"IP(s)"** field
3. Copy the IP address (e.g., `162.230.144.169`)

This is the **LoadBalancer IP** assigned to your validator deployment.

### Step 2: Update Monitoring Deployment

**Before deploying, update the `METAL_NODE_URL` environment variable:**

1. Open `deploy.yaml` from this repository
2. Find the line: `METAL_NODE_URL=http://YOUR_VALIDATOR_IP_HERE:9650`
3. Replace `YOUR_VALIDATOR_IP_HERE` with your actual validator IP
4. Save the file

**Or set via Akash Console after deployment:**
- Go to Akash Console → Your Monitoring Deployment → Environment Variables
- Update `METAL_NODE_URL` = `http://<YOUR_VALIDATOR_IP>:9650`

### Step 3: Deploy Monitoring

1. Go to **Akash Console** → Deploy
2. Click **"Upload SDL"**
3. Upload your updated `deploy.yaml`
4. Choose a provider with `ip-lease: true` (recommended)
5. Accept bid and deploy

### Step 4: Access Grafana

**After deployment (2-3 minutes):**

1. Go to **Akash Console** → Your Monitoring Deployment
2. Look for **"IP(s)"** field (shows Grafana IP)
3. Copy the IP address (e.g., `162.230.144.169`)
4. Access Grafana at: `http://<GRAFANA_IP>:3000`
   - Example: `http://162.230.144.169:3000`

**Login Credentials:**
- **Username**: `admin`
- **Password**: `password123`

### Step 5: Configure Prometheus Data Source (REQUIRED!)

**⚠️ IMPORTANT: Dashboards won't work until you configure the Prometheus data source!**

1. **In Grafana (after login):**
   - Click the **gear icon** (⚙️) in the left sidebar → **"Data Sources"**
   - Click **"Add data source"**
   - Select **"Prometheus"**

2. **Configure the URL:**
   - **Option 1**: Try service name first: `http://prometheus:9090`
     - This uses the service name (works within same Akash deployment)
   - **Option 2**: If service name doesn't work, use forwarded port:
     - Get from Akash Console → Monitoring Deployment → Leases tab
     - Find "prometheus" Group → Forwarded Ports (e.g., `9090:31801`)
     - URL: `http://<YOUR_PROVIDER>:<PROMETHEUS_FORWARDED_PORT>`
     - Example: `http://provider.akash.tagus.host:31801`

3. **Save:**
   - Click **"Save & Test"**
   - Should show: ✅ **"Data source is working"**

### Step 6: Import Metal Dashboards

1. **Go to Dashboards → Import** (plus icon → Import)
2. **Import from [Metal Monitoring Repository](https://github.com/MetalBlockchain/metal-monitoring)**:
   - Dashboard IDs available in the repository
   - Or import JSON files directly
3. **Select Prometheus data source** when importing
4. **Dashboards should populate with data**

## What You Get

### Services

1. **Prometheus** - Scrapes and stores metrics from Metal node
   - Scrapes from `/ext/metrics` endpoint
   - 15-second scrape interval
   - Accessible on port 9090

2. **Grafana** - Visualizes metrics in dashboards (main UI)
   - Auto-configures Prometheus data source
   - Accessible on port 3000
   - Default credentials: admin / password123

3. **Node Exporter** - System metrics (CPU, memory, disk)
   - Container resource monitoring
   - Exposes metrics on port 9100

### Monitoring Capabilities

- **Node Health**: Peer count, sync status, block height, uptime
- **Performance**: CPU usage, memory usage, disk I/O, network traffic
- **Blockchain Metrics**: Transaction throughput, block production rate, validator participation, staking status

### Available Dashboards

Metal provides several pre-built dashboards:

1. **Metal Main Dashboard** - Overview of node health
2. **C-Chain Dashboard** - C-Chain specific metrics
3. **P-Chain Dashboard** - Platform chain metrics
4. **X-Chain Dashboard** - Exchange chain metrics
5. **Network Dashboard** - Network connectivity metrics
6. **Machine Dashboard** - System resource usage

## Access Methods

### Direct IP Access (Recommended - v2.1)

- **Grafana**: `http://<GRAFANA_IP>:3000`
- **Prometheus**: `http://<GRAFANA_IP>:9090` (if accessible)
- **Advantage**: Bypasses provider port forwarding restrictions
- **Requires**: Provider with `ip-lease: true`

### Forwarded Ports (Alternative - v2.0)

If you need to use forwarded ports instead:

1. Get forwarded ports from Akash Console → Leases tab
2. Get provider hostname from deployment details
3. Access via: `http://<PROVIDER>:<FORWARDED_PORT>`

**Note**: Some providers block external access to forwarded ports. Use IP lease version (v2.1) for more reliable access.

## Verify Monitoring is Working

**Check Prometheus targets:**
- Go to: `http://<PROMETHEUS_URL>/targets`
- You should see 3 targets:
  - ✅ `prometheus` (UP)
  - ✅ `metalgo` (UP) - This is your Metal node
  - ✅ `node-exporter` (UP)

If `metalgo` shows DOWN:
- Verify `METAL_NODE_URL` is correct
- Check your validator is still running
- Verify the IP is accessible from the monitoring deployment

## Troubleshooting

### Grafana Not Accessible

- Verify IP is assigned in Akash Console → Deployment → "IP(s)" field
- Check Grafana is running: Shell tab → `curl http://localhost:3000/api/health`
- Verify provider supports IP leases

### Prometheus Data Source Not Working

- Check Prometheus is running: Shell tab → `curl http://localhost:9090`
- Verify Prometheus URL in Grafana settings
- Try service name: `http://prometheus:9090`
- Try forwarded port URL: `http://<PROVIDER>:<PROMETHEUS_PORT>`
- Check Prometheus targets: `http://<PROMETHEUS_URL>/targets`

### Dashboards Empty

- Verify Prometheus data source is configured and working
- Check Prometheus is scraping Metal node (`/targets` page)
- Verify `METAL_NODE_URL` is correct
- Check Metal node metrics endpoint: `curl http://<VALIDATOR_IP>:9650/ext/metrics`
- Import Metal dashboards manually if needed

### Prometheus Can't Scrape Metal Node

- Verify `METAL_NODE_URL` format: `http://IP:9650` (no trailing slash)
- Check Metal node metrics endpoint: `curl http://<VALIDATOR_IP>:9650/ext/metrics`
- Ensure both deployments are on the same provider (or use public IP)
- Verify Metal node is still running

### Grafana External URL Not Working (Connection Refused)

**Symptom:**
- Grafana health check works from Shell tab: `curl http://localhost:3000/api/health` returns JSON
- External URL fails: Connection refused

**Root Cause:**
- Provider-side networking restriction - Some providers block external access to forwarded ports

**Solutions:**
1. **Use IP Lease Version (v2.1)**: Deploy with `ip-lease: true` for direct IP access
2. **Deploy to Different Provider**: Some providers allow external access
3. **Use Akash Console's Built-in Port Forwarding**: If available
4. **Contact Provider**: Request external port forwarding access

## Security Considerations

⚠️ **IMPORTANT**: These services are NOT secured by default!

**Default Credentials:**
- Grafana: `admin` / `password123`

**Security Recommendations:**
1. **Change default password** in Grafana UI (Configuration → Users → Change Password)
2. **Use VPN or IP whitelisting** if exposing to internet
3. **Consider using Akash's private networking** features
4. **Enable additional authentication** in Grafana settings
5. **Use HTTPS** if exposing publicly (requires reverse proxy setup)

## Alternative: Simple Health Checks

If you don't need full monitoring, you can use simple health checks:

```bash
# Check node health
curl http://<YOUR_NODE_IP>:9650/ext/health

# Get peer count
curl -X POST http://<YOUR_NODE_IP>:9650/ext/info \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"info.peers","params":{},"id":1}'

# Get node status
curl -X POST http://<YOUR_NODE_IP>:9650/ext/info \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"info.getNodeID","params":{},"id":1}'
```

## Resources

- [Metal Monitoring Repository](https://github.com/MetalBlockchain/metal-monitoring)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Metal Metrics API](https://docs.metalblockchain.org/apis/avalanchego/apis/info#info-metrics)
- [Akash Documentation](https://docs.akash.network/)

## Support

- **Email**: cerebro@cerebro.host
- **Metal Discord**: https://discord.gg/B2QDmgf
- **Akash Discord**: #support channel

---

**Need Help?**
- Review the troubleshooting section above
- Check [Akash Documentation](https://docs.akash.network/) for deployment issues
- Join [Metal Discord](https://discord.gg/B2QDmgf) for support
