################ Adding my subdomain for the cookie app ####################
To update your DNS records for the domain `gm-nig-ltd.tech` using Cloudflare and point a subdomain (e.g., `cookieclicker.gm-nig-ltd.tech`) to an external IP address (e.g., `212.2.240.68`) of your Kubernetes ingress controller, follow these steps. This process involves logging into your Cloudflare account, navigating to the DNS management section for your domain, and creating an A record.

### Step 1: Log into Cloudflare

1. **Open your web browser** and go to [Cloudflare's website](https://www.cloudflare.com/).
2. **Log in** to your Cloudflare account by clicking on the **Log In** button at the top right corner and entering your credentials.

### Step 2: Navigate to Your Domain's DNS Management Page

1. Once logged in, you'll see the dashboard with a list of your domains.
2. **Select your domain** `gm-nig-ltd.tech` by clicking on it. This action takes you to the domain's overview page.
3. From the top menu options, **click on the "DNS"** tab to access the DNS management page for your domain.

### Step 3: Create an A Record

1. On the DNS management page, you'll see a section where you can add new DNS records.
2. To create a new A record, look for a button or link that says **"Add record"** or **"+ Add"** within the DNS records section.
3. In the **Type** field, select **A** from the dropdown menu. This specifies that you're creating an A record.
4. In the **Name** field, enter the subdomain you want to use. For pointing directly to `gm-nig-ltd.tech`, you can enter `@` to represent the root domain. For a subdomain like `cookieclicker.gm-nig-ltd.tech`, just enter `cookieclicker`.
5. In the **IPv4 address** field, enter the external IP address of your ingress controller, which is `212.2.240.68`.
6. Depending on your needs, you can adjust the **TTL** (Time To Live) value, but the default value is typically fine for most purposes.
7. **Save** the record by clicking on the **Save** button or its equivalent.

### Step 4: Verify the DNS Record

After adding the A record, it might take some time for the changes to propagate across the internet due to DNS caching. You can check if the record is correctly pointing to the external IP by using DNS lookup tools like `nslookup` or `dig`.

For example, to use `nslookup`:

```bash
nslookup cookieclicker.gm-nig-ltd.tech
```

This command should return the external IP address `212.2.240.68` as one of the addresses for `cookieclicker.gm-nig-ltd.tech`.

### Additional Tips

- **Propagation Time**: DNS changes can take up to 48 hours to fully propagate, but they often take effect much sooner.
- **Cloudflare Proxies**: If you're using Cloudflare's proxy (the cloud icon next to the DNS record is orange), the actual IP address will be hidden behind Cloudflare's reverse proxy for additional security and performance features. If you want to bypass Cloudflare's proxy and reveal the actual IP, you can click the cloud icon so that it turns grey, which disables the proxy.
- **SSL/TLS Configuration**: Ensure that your Cloudflare SSL/TLS encryption mode matches your setup requirements and is compatible with your ingress controller's configuration.

By following these steps, you will have successfully pointed your subdomain `cookieclicker.gm-nig-ltd.tech` to the external IP address of your Kubernetes ingress controller, allowing traffic to be routed correctly.