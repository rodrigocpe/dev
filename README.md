https://hands-on.cloud/managing-amazon-api-gateway-using-terraform/



resource "aws_api_gateway_rest_api" "rest_api" {
  name = var.rest_api_name
}


##health
#########################################################################################################################################################
#########################################################################################################################################################
#########################################################################################################################################################
#########################################################################################################################################################

#health/GET
resource "aws_api_gateway_resource" "rest_api_method_health" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  parent_id   = aws_api_gateway_rest_api.rest_api.root_resource_id
  path_part   = "health"
}
resource "aws_api_gateway_method" "rest_api_method_health" {
  rest_api_id   = aws_api_gateway_rest_api.rest_api.id
  resource_id   = aws_api_gateway_resource.rest_api_method_health.id
  http_method   = "GET"
  authorization = "NONE"
}
resource "aws_api_gateway_method_response" "rest_api_method_response_200_health" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_method_health.id
  http_method = aws_api_gateway_method.rest_api_method_health.http_method
  status_code = "200"
}
resource "aws_api_gateway_integration" "rest_api_method_integration_health" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_method_health.id
  http_method = aws_api_gateway_method.rest_api_method_health.http_method
  type        = "MOCK"
}
resource "aws_api_gateway_integration_response" "rest_api_integration_response_health" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_method_health.id
  http_method = aws_api_gateway_method.rest_api_method_health.http_method
  status_code = aws_api_gateway_method_response.rest_api_method_response_200_health.status_code

  depends_on = [
    aws_api_gateway_method_response.rest_api_method_response_200_health,
    aws_api_gateway_integration.rest_api_method_integration_health,
  ]
}
resource "aws_lambda_permission" "api_gateway_lambda" {
  statement_id  = "AllowExecutionFromAPIGateway"
  action        = "lambda:InvokeFunction"
  function_name = var.lambda_function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "arn:aws:execute-api:${var.api_gateway_region}:${var.api_gateway_account_id}:${aws_api_gateway_rest_api.rest_api.id}/*/${aws_api_gateway_method.rest_api_method_health.http_method}${aws_api_gateway_resource.rest_api_method_health.path}"
}
#health/OPTIONS
resource "aws_api_gateway_method" "options_method_health" {
  rest_api_id   = aws_api_gateway_rest_api.rest_api.id
  resource_id   = aws_api_gateway_resource.rest_api_method_health.id
  http_method   = "OPTIONS"
  authorization = "NONE"
}
resource "aws_api_gateway_method_response" "options_200_health" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_method_health.id
  http_method = aws_api_gateway_method.options_method_health.http_method
  status_code = "200"
  response_parameters = {
    "method.response.header.Access-Control-Allow-Origin"  = true
    "method.response.header.Access-Control-Allow-Headers" = true
    "method.response.header.Access-Control-Allow-Methods" = true

  }


  depends_on = [aws_api_gateway_method.options_method_health]
}
resource "aws_api_gateway_integration" "options_integration_health" {
  rest_api_id          = aws_api_gateway_rest_api.rest_api.id
  resource_id          = aws_api_gateway_resource.rest_api_method_health.id
  http_method          = aws_api_gateway_method.options_method_health.http_method
  type                 = "MOCK"
  content_handling     = "CONVERT_TO_TEXT"
  passthrough_behavior = "NEVER"

  depends_on = [aws_api_gateway_method.options_method_health]
}
resource "aws_api_gateway_integration_response" "options_integration_response_health" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_method_health.id
  http_method = aws_api_gateway_method.options_method_health.http_method
  status_code = aws_api_gateway_method_response.options_200_health.status_code

  response_parameters = {
    "method.response.header.Access-Control-Allow-Origin"  = "'*'"
    "method.response.header.Access-Control-Allow-Headers" = "'Content-Type,Authorization,X-Alg-Dl-File-Path,X-Alg-Dl-Remote-Account-Id,X-Amz-Date,X-Api-Key,X-Amz-Security-Token'"
    "method.response.header.Access-Control-Allow-Methods" = "'HEAD,OPTIONS,PATCH,POST,GET'"
  }

  depends_on = [
    aws_api_gateway_method_response.options_200_health,
    aws_api_gateway_integration.options_integration_health,
  ]
}

#########################################################################################################################################################
#########################################################################################################################################################
#########################################################################################################################################################
#########################################################################################################################################################

#objects/post
resource "aws_api_gateway_resource" "rest_api_resource_objects" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  parent_id   = aws_api_gateway_rest_api.rest_api.root_resource_id
  path_part   = "objects"
}
resource "aws_api_gateway_method" "rest_api_method_objects" {
  rest_api_id   = aws_api_gateway_rest_api.rest_api.id
  resource_id   = aws_api_gateway_resource.rest_api_resource_objects.id
  http_method   = "POST"
  authorization = "NONE"
  request_parameters = {

    "method.request.header.Content-Type"              = true
    "method.request.header.x-alg-dl-file-path"        = true
    "method.request.header.x-alg-dl-custom-file-name" = false
    "method.request.header.x-alg-dl-date"             = false
    "method.request.header.x-alg-dl-custom-bucket"    = false
    "method.request.header.x-alg-dl-ingestion-mode"   = false
  }
}
resource "aws_api_gateway_method_response" "rest_api_method_response_200_objects" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_resource_objects.id
  http_method = aws_api_gateway_method.rest_api_method_objects.http_method
  status_code = "200"
}
resource "aws_api_gateway_integration" "rest_api_method_integration_objects" {
  rest_api_id             = aws_api_gateway_rest_api.rest_api.id
  resource_id             = aws_api_gateway_resource.rest_api_resource_objects.id
  http_method             = aws_api_gateway_method.rest_api_method_objects.http_method
  integration_http_method = "POST"
  type                    = "AWS_PROXY"
  uri                     = var.lambda_function_arn
}

#object/OPTIONS
resource "aws_api_gateway_method" "options_method_objects" {
  rest_api_id   = aws_api_gateway_rest_api.rest_api.id
  resource_id   = aws_api_gateway_resource.rest_api_resource_objects.id
  http_method   = "OPTIONS"
  authorization = "NONE"
}
resource "aws_api_gateway_method_response" "options_200_objects" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_resource_objects.id
  http_method = aws_api_gateway_method.options_method_objects.http_method
  status_code = "200"
  response_parameters = {
    "method.response.header.Access-Control-Allow-Origin"  = true
    "method.response.header.Access-Control-Allow-Headers" = true
    "method.response.header.Access-Control-Allow-Methods" = true

  }
}
resource "aws_api_gateway_integration" "options_integration_objects" {
  rest_api_id      = aws_api_gateway_rest_api.rest_api.id
  resource_id      = aws_api_gateway_resource.rest_api_resource_objects.id
  http_method      = aws_api_gateway_method.options_method_objects.http_method
  type             = "MOCK"
  content_handling = "CONVERT_TO_TEXT"
}
resource "aws_api_gateway_integration_response" "options_integration_response_objects" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_resource_objects.id
  http_method = aws_api_gateway_method.options_method_objects.http_method
  status_code = aws_api_gateway_method_response.options_200_objects.status_code
}

#########################################################################################################################################################
#########################################################################################################################################################
#########################################################################################################################################################
#########################################################################################################################################################

#topics/POST
resource "aws_api_gateway_resource" "rest_api_method_topics" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  parent_id   = aws_api_gateway_rest_api.rest_api.root_resource_id
  path_part   = "topics"
}
resource "aws_api_gateway_method" "rest_api_method_topics" {
  rest_api_id   = aws_api_gateway_rest_api.rest_api.id
  resource_id   = aws_api_gateway_resource.rest_api_method_topics.id
  http_method   = "POST"
  authorization = "NONE"
}
resource "aws_api_gateway_method_response" "rest_api_method_response_200_topics" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_method_topics.id
  http_method = aws_api_gateway_method.rest_api_method_topics.http_method
  status_code = "200"
}
resource "aws_api_gateway_integration" "rest_api_method_integration_topics" {
  rest_api_id             = aws_api_gateway_rest_api.rest_api.id
  resource_id             = aws_api_gateway_resource.rest_api_method_topics.id
  http_method             = aws_api_gateway_method.rest_api_method_topics.http_method
  integration_http_method = "POST"
  type                    = "AWS_PROXY"
  uri                     = var.lambda_function_arn
}
resource "aws_api_gateway_integration_response" "rest_api_integration_response_topics" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_method_topics.id
  http_method = aws_api_gateway_method.rest_api_method_topics.http_method
  status_code = aws_api_gateway_method_response.rest_api_method_response_200_topics.status_code
}

#topics/OPTIONS
resource "aws_api_gateway_method" "options_method_topics" {
  rest_api_id   = aws_api_gateway_rest_api.rest_api.id
  resource_id   = aws_api_gateway_resource.rest_api_method_topics.id
  http_method   = "OPTIONS"
  authorization = "NONE"
}
resource "aws_api_gateway_method_response" "options_200_topics" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_method_topics.id
  http_method = aws_api_gateway_method.options_method_topics.http_method
  status_code = "200"
  response_parameters = {
    "method.response.header.Access-Control-Allow-Origin"  = true
    "method.response.header.Access-Control-Allow-Headers" = true
    "method.response.header.Access-Control-Allow-Methods" = true
  }

}
resource "aws_api_gateway_integration" "options_integration_topics" {
  rest_api_id          = aws_api_gateway_rest_api.rest_api.id
  resource_id          = aws_api_gateway_resource.rest_api_method_topics.id
  http_method          = aws_api_gateway_method.options_method_topics.http_method
  type                 = "MOCK"
  content_handling     = "CONVERT_TO_TEXT"
  passthrough_behavior = "WHEN_NO_MATCH"
  # (WHEN_NO_MATCH, WHEN_NO_TEMPLATES, NEVER). Required if request_templates is used.
}
resource "aws_api_gateway_integration_response" "options_integration_response_topics" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_method_topics.id
  http_method = aws_api_gateway_method.options_method_topics.http_method
  status_code = aws_api_gateway_method_response.options_200_topics.status_code
  # response_parameters = {
  #   method.response.header.Access-Control-Allow-Methods : "'HEAD,OPTIONS,PATCH,POST'"
  #   method.response.header.Access-Control-Allow-Headers : "'Content-Type,Authorization,X-Amz-Date,X-Api-Key,X-Amz-Security-Token,x-alg-dl-file-path,x-alg-dl-custom-file-name,x-alg-dl-custom-bucket,x-alg-dl-date,x-alg-dl-ingestion-mode'"
  #   method.response.header.Access-Control-Allow-Origin : "'*'"

  # }
  #response_parameters = { "method.response.header.X-Some-Header" = "integration.response.header.X-Some-Other-Header" }

}

#topics/{topicid}/GET
resource "aws_api_gateway_resource" "rest_api_resource_topics_child1" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  parent_id   = aws_api_gateway_rest_api.rest_api.root_resource_id
  path_part   = "{topicid}"
}

resource "aws_api_gateway_method" "rest_api_method_topics_child1" {
  rest_api_id   = aws_api_gateway_rest_api.rest_api.id
  resource_id   = aws_api_gateway_resource.rest_api_resource_topics_child1.id
  http_method   = "GET"
  authorization = "NONE"
  request_parameters = {
    "method.request.header.Content-Type" = true
  }
}
resource "aws_api_gateway_method_response" "rest_api_method_response_200_topics_child1" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_resource_topics_child1.id
  http_method = aws_api_gateway_method.rest_api_method_topics_child1.http_method
  status_code = "200"
}
resource "aws_api_gateway_integration" "rest_api_method_integration_topics_child1" {
  rest_api_id             = aws_api_gateway_rest_api.rest_api.id
  resource_id             = aws_api_gateway_resource.rest_api_resource_topics_child1.id
  http_method             = aws_api_gateway_method.rest_api_method_topics_child1.http_method
  integration_http_method = "POST"
  type                    = "AWS_PROXY"
  uri                     = var.lambda_function_arn
}
resource "aws_api_gateway_integration_response" "rest_api_integration_response_topics_child1" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  resource_id = aws_api_gateway_resource.rest_api_resource_topics_child1.id
  http_method = aws_api_gateway_method.rest_api_method_topics_child1.http_method
  status_code = aws_api_gateway_method_response.rest_api_method_response_200_topics_child1.status_code

  depends_on = [
    aws_api_gateway_method_response.rest_api_method_response_200_topics_child1,
    aws_api_gateway_integration.rest_api_method_integration_topics_child1,
  ]
}




resource "aws_api_gateway_deployment" "rest_api_deployment" {
  rest_api_id = aws_api_gateway_rest_api.rest_api.id
  lifecycle {
    create_before_destroy = true
  }
}


resource "aws_api_gateway_stage" "rest_api_stage" {
  deployment_id = aws_api_gateway_deployment.rest_api_deployment.id
  rest_api_id   = aws_api_gateway_rest_api.rest_api.id
  stage_name    = var.rest_api_stage_name
}
