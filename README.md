# terraform-aws-waf

[![Lint Status](https://github.com/DNXLabs/terraform-aws-waf/workflows/Lint/badge.svg)](https://github.com/DNXLabs/terraform-aws-waf/actions)
[![LICENSE](https://img.shields.io/github/license/DNXLabs/terraform-aws-waf)](https://github.com/DNXLabs/terraform-aws-waf/blob/master/LICENSE)

This terraform module creates two type of WAFv2 Web ACL rules:
  - CLOUDFRONT is a Global rule used in CloudFront Distribution only
  - REGIONAL rules can be used in ALB, API Gateway or AppSync GraphQL API

Follow a commum list of Web ACL rules that can be used by this module and how to setup it, also a link of the documentation with a full list of AWS WAF Rules, you need to use the "Name" of the Rule Groups and take care with WCUs, it's why Web ACL rules can't exceed 1500 WCUs.

  - byte_match_statement
  - geo_match_statement
  - ip_set_reference_statement
  - managed_rule_group_statement
    - AWSManagedRulesCommonRuleSet
    - AWSManagedRulesAmazonIpReputationList
    - AWSManagedRulesKnownBadInputsRuleSet
    - AWSManagedRulesSQLiRuleSet
    - AWSManagedRulesLinuxRuleSet
    - AWSManagedRulesUnixRuleSet
  - rate_based_statement
  - regex_pattern_set_reference_statement
  - size_constraint_statement
  - sqli_match_statement
  - xss_match_statement

## Usage

```hcl

module "terraform_aws_wafv2_global" {
  source   = "git::https://github.com/DNXLabs/terraform-aws-waf.git?ref=1.1.0"
  for_each = { for rule in try(local.workspace.wafv2_global.rules, []) : rule.global_rule => rule }

  waf_cloudfront_enable = try(each.value.waf_cloudfront_enable, false)
  web_acl_id            = try(each.value.web_acl_id, "") # Optional WEB ACLs (WAF) to attach to CloudFront
  global_rule           = try(each.value.global_rule, [])
  scope                 = each.value.scope
  default_action        = try(each.value.default_action, "block")

  ### Log Configuration
  logs_enable             = try(each.value.logs_enable, false)
  logs_retension          = try(each.value.logs_retension, 90)
  logging_redacted_fields = try(each.value.logging_redacted_fields, [])
  logging_filter          = try(each.value.logging_filter, [])

  ### Statement Rules
  byte_match_statement_rules                  = try(each.value.byte_match_statement_rules, [])
  geo_match_statement_rules                   = try(each.value.geo_match_statement_rules, [])
  ip_set_reference_statement_rules            = try(each.value.ip_set_reference_statement_rules, [])
  managed_rule_group_statement_rules          = try(each.value.managed_rule_group_statement_rules, [])
  rate_based_statement_rules                  = try(each.value.rate_based_statement_rules, [])
  regex_pattern_set_reference_statement_rules = try(each.value.regex_pattern_set_reference_statement_rules, [])
  size_constraint_statement_rules             = try(each.value.size_constraint_statement_rules, [])
  sqli_match_statement_rules                  = try(each.value.sqli_match_statement_rules, [])
  xss_match_statement_rules                   = try(each.value.xss_match_statement_rules, [])
}

data "aws_wafv2_web_acl" "web_acl_arn" {
# count = local.workspace.wafv2.global.waf_cloudfront_web_acl_enable ? 1 : 0
depends_on = [module.terraform_aws_wafv2_global]
provider = aws.us-east-1
  name  = "waf-${local.workspace.wafv2.global.acls.global_rule_name}"
  scope = "CLOUDFRONT"
}

module "terraform_aws_wafv2_regional" {
  source   = "git::https://github.com/DNXLabs/terraform-aws-waf.git?ref=1.1.0"
  for_each = { for rule in try(local.workspace.wafv2_regional.rules, []) : rule.regional_rule => rule }

  waf_regional_enable = try(each.value.waf_regional_enable, false)
  associate_waf       = try(each.value.associate_waf, false)
  regional_rule       = try(each.value.regional_rule, [])
  scope               = each.value.scope
  resource_arn        = try(each.value.resource_arn, [])
  default_action      = try(each.value.default_action, "block")

  ### Log Configuration
  logs_enable             = try(each.value.logs_enable, false)
  logs_retension          = try(each.value.logs_retension, 90)
  logging_redacted_fields = try(each.value.logging_redacted_fields, [])
  logging_filter          = try(each.value.logging_filter, [])

  ### Statement Rules
  byte_match_statement_rules                  = try(each.value.byte_match_statement_rules, [])
  geo_match_statement_rules                   = try(each.value.geo_match_statement_rules, [])
  ip_set_reference_statement_rules            = try(each.value.ip_set_reference_statement_rules, [])
  managed_rule_group_statement_rules          = try(each.value.managed_rule_group_statement_rules, [])
  rate_based_statement_rules                  = try(each.value.rate_based_statement_rules, [])
  regex_pattern_set_reference_statement_rules = try(each.value.regex_pattern_set_reference_statement_rules, [])
  size_constraint_statement_rules             = try(each.value.size_constraint_statement_rules, [])
  sqli_match_statement_rules                  = try(each.value.sqli_match_statement_rules, [])
  xss_match_statement_rules                   = try(each.value.xss_match_statement_rules, [])
}q
```

<!--- BEGIN_TF_DOCS --->

## Requirements

| Name | Version |
|------|---------|
| terraform | >= 0.13.0 |

## Providers

| Name | Version |
|------|---------|
| aws | n/a |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| alb\_arn | ARN of the ALB to be associated with the WAFv2 ACL. | `string` | `""` | no |
| api\_gateway\_arn | ARN of the API Gateway to be associated with the WAFv2 ACL. | `string` | `""` | no |
| associate\_alb | Whether to associate an ALB with the WAFv2 ACL. | `bool` | `false` | no |
| filtered\_header\_rule | HTTP header to filter . Currently supports a single header type and multiple header values. | <pre>object({<br>    header_types = list(string)<br>    priority     = number<br>    header_value = string<br>    action       = string<br>  })</pre> | <pre>{<br>  "action": "block",<br>  "header_types": [],<br>  "header_value": "",<br>  "priority": 1<br>}</pre> | no |
| global\_rule | Cloudfront WAF Rule Name | `string` | `""` | no |
| ip\_rate\_based\_rule | A rate-based rule tracks the rate of requests for each originating IP address, and triggers the rule action when the rate exceeds a limit that you specify on the number of requests in any 5-minute time span | <pre>object({<br>    name     = string<br>    priority = number<br>    limit    = number<br>    action   = string<br>  })</pre> | `null` | no |
| ip\_rate\_url\_based\_rules | A rate and url based rules tracks the rate of requests for each originating IP address, and triggers the rule action when the rate exceeds a limit that you specify on the number of requests in any 5-minute time span | <pre>list(object({<br>    name                  = string<br>    priority              = number<br>    limit                 = number<br>    action                = string<br>    search_string         = string<br>    positional_constraint = string<br>  }))</pre> | `[]` | no |
| ip\_sets\_rule | A rule to detect web requests coming from particular IP addresses or address ranges. | <pre>list(object({<br>    name       = string<br>    priority   = number<br>    ip_set_arn = string<br>    action     = string<br>  }))</pre> | `[]` | no |
| managed\_rules | List of Managed WAF rules. | <pre>list(object({<br>    name            = string<br>    priority        = number<br>    override_action = string<br>    excluded_rules  = list(string)<br>  }))</pre> | <pre>[<br>  {<br>    "excluded_rules": [],<br>    "name": "AWSManagedRulesCommonRuleSet",<br>    "override_action": "none",<br>    "priority": 10<br>  },<br>  {<br>    "excluded_rules": [],<br>    "name": "AWSManagedRulesAmazonIpReputationList",<br>    "override_action": "none",<br>    "priority": 20<br>  },<br>  {<br>    "excluded_rules": [],<br>    "name": "AWSManagedRulesKnownBadInputsRuleSet",<br>    "override_action": "none",<br>    "priority": 30<br>  },<br>  {<br>    "excluded_rules": [],<br>    "name": "AWSManagedRulesSQLiRuleSet",<br>    "override_action": "none",<br>    "priority": 40<br>  },<br>  {<br>    "excluded_rules": [],<br>    "name": "AWSManagedRulesLinuxRuleSet",<br>    "override_action": "none",<br>    "priority": 50<br>  },<br>  {<br>    "excluded_rules": [],<br>    "name": "AWSManagedRulesUnixRuleSet",<br>    "override_action": "none",<br>    "priority": 60<br>  }<br>]</pre> | no |
| regional\_rule | Regional WAF Rules for ALB and API Gateway | `string` | `""` | no |
| scope | The scope of this Web ACL. Valid options: CLOUDFRONT, REGIONAL(ALB). | `string` | n/a | yes |
| waf\_cloudfront\_enable | Enable WAF for Cloudfront distribution | `bool` | `false` | no |
| waf\_regional\_enable | Enable WAFv2 to ALB, API Gateway or AppSync GraphQL API | `bool` | `false` | no |
| wafv2\_managed\_block\_rule\_groups | List of WAF V2 managed rule groups, set to block | `list(string)` | `[]` | no |
| wafv2\_managed\_rule\_groups | List of WAF V2 managed rule groups, set to count | `list(string)` | <pre>[<br>  "AWSManagedRulesCommonRuleSet"<br>]</pre> | no |
| wafv2\_rate\_limit\_rule | The limit on requests per 5-minute period for a single originating IP address (leave 0 to disable) | `number` | `0` | no |
| web\_acl\_id | Specify a web ACL ARN to be associated in CloudFront Distribution / # Optional WEB ACLs (WAF) to attach to CloudFront | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| web\_acl\_arn | The ARN of the WAFv2 WebACL. |
| web\_acl\_capacity\_cloudfront | The web ACL capacity units (WCUs) currently being used by this web ACL. |
| web\_acl\_capacity\_regional | The web ACL capacity units (WCUs) currently being used by this web ACL. |
| web\_acl\_id | The ID of the WAFv2 WebACL. |
| web\_acl\_name\_cloudfront | The name of the WAFv2 WebACL. |
| web\_acl\_name\_regional | The name of the WAFv2 WebACL. |
| web\_acl\_visibility\_config\_name\_cloudfront | The web ACL visibility config name |
| web\_acl\_visibility\_config\_name\_regional | The web ACL visibility config name |

<!--- END_TF_DOCS --->


## Authors

Module managed by [DNX Solutions](https://github.com/DNXLabs).

## License

Apache 2 Licensed. See [LICENSE](https://github.com/DNXLabs/terraform-aws-waf/blob/master/LICENSE) for full details.