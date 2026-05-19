# Netify BT-Authorised Reseller MCP Server

A Model Context Protocol (MCP) server that exposes the [Netify BT-Authorised Reseller Programme](https://www.netify.co.uk/resell/bt-business-services/) as a callable surface for AI agents.

UK Limited Companies (MSPs, IT consultants, telecom resellers, EPOS providers, cybersecurity firms) can use this server to:

- Query programme details (route, fees, onboarding length)
- Score themselves against the five BT-Authorised Reseller eligibility criteria
- Model qualitative commission for a given sales mix
- List BT resellable products with full technical details
- Read BT compliance requirements (workspace audit, brand rules, Ofcom General Conditions)
- Construct a prefilled application URL ready for submission

## Endpoint

```
POST https://www.netify.co.uk/resell/bt-business-services/mcp/
Content-Type: application/json
```

Transport: **streamable-http** (synchronous JSON-RPC 2.0 over HTTP POST).
Auth: **none**.
CORS: open (`Access-Control-Allow-Origin: *`).

A `GET` on the same URL returns a server-info banner.

## Methods

| Method | Purpose |
| --- | --- |
| `initialize` | Standard MCP handshake. Returns `protocolVersion`, `capabilities`, `serverInfo`. |
| `tools/list` | Returns the full list of tools with input schemas. |
| `tools/call` | Invokes a named tool with arguments. |
| `ping` | Returns an empty result. Used by clients to verify liveness. |
| `notifications/*` | Accepted, no body returned. |

## Tools

| Tool | Purpose |
| --- | --- |
| `get_programme_info` | Programme metadata (route, fees=£0, joining target=0, partner details). |
| `list_products` | All seven resellable BT Business products. Optional `category` filter. |
| `get_product` | Full details for one product by key (`fttp`, `sogea`, `cve`, `badr`, `btnet`, `sdwan`, `sase`). |
| `check_eligibility` | Scores a prospective reseller 0-100 against the five BT dimensions. Returns blockers and next steps. |
| `estimate_commission` | Qualitative monthly and annual run-rate commission estimate (ranges and formula descriptions, never exact rates). |
| `build_application_url` | Constructs a prefilled reseller application URL. |
| `list_compliance_requirements` | The five compliance checks BT actually performs. |

Full input schemas are returned by `tools/list`. Structured data is also available without invoking the MCP server at [https://www.netify.co.uk/resell/bt-business-services/data.json](https://www.netify.co.uk/resell/bt-business-services/data.json).

## Quick test

```bash
# Tools list
curl -X POST https://www.netify.co.uk/resell/bt-business-services/mcp/ \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}'

# Eligibility check
curl -X POST https://www.netify.co.uk/resell/bt-business-services/mcp/ \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc":"2.0","id":2,"method":"tools/call",
    "params":{
      "name":"check_eligibility",
      "arguments":{
        "is_uk_limited_company":true,
        "has_custom_domain_website":true,
        "has_business_email_on_domain":true,
        "has_dedicated_physical_workspace":true,
        "has_defined_route_to_market":true,
        "business_type":"MSP"
      }
    }
  }'
```

## Use with Claude Desktop, Cursor, VS Code

Add to your MCP client config:

```json
{
  "mcpServers": {
    "netify-bt-reseller": {
      "type": "streamable-http",
      "url": "https://www.netify.co.uk/resell/bt-business-services/mcp/"
    }
  }
}
```

## Discovery

Agents can discover this server from the host page:

```html
<link rel="mcp_endpoint" type="application/json"
      title="Netify MCP Server (JSON-RPC 2.0 over HTTP POST)"
      href="https://www.netify.co.uk/resell/bt-business-services/mcp/">
```

A static JSON representation of the underlying programme data is also linked:

```html
<link rel="alternate" type="application/json"
      title="Machine-readable programme data"
      href="https://www.netify.co.uk/resell/bt-business-services/data.json">
```

## Commercial transparency

The `estimate_commission` tool deliberately returns qualitative monthly and annual *ranges*, not exact rates. Exact commercials (rates, splits, tail length) are confirmed per signed Netify reseller agreement at application stage. This is a deliberate design choice — the rates are commercial-in-confidence between Netify and its resellers.

## About Netify

[Netify Group Limited](https://www.netify.co.uk/) is a UK-registered company (Company No. 07087612), BT Authorised Partner since 2012. Operates [netify.co.uk](https://www.netify.co.uk/) — an SD-WAN and SASE marketplace with RFP builder — and recruits Authorised Resellers of BT through the [BT-Authorised Reseller Programme](https://www.netify.co.uk/resell/bt-business-services/).

Registered office: Moor Hall Barn, Workhouse Lane, Melton Constable, Norfolk, NR24 2BE, England.

Contact: [support@netify.com](mailto:support@netify.com) — +44 (0)333 202 1011

## License

MIT. See `LICENSE`.
