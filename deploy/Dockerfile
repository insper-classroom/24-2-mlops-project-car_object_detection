FROM public.ecr.aws/lambda/python:3.12

# Copy requirements.txt
COPY deploy/requirements/deploy_requirements.txt ${LAMBDA_TASK_ROOT}

# Copy function code
COPY deploy/lambda_function.py ${LAMBDA_TASK_ROOT}

# Install the specified packages
RUN pip install -r deploy_requirements.txt

# Set the CMD to your handler (could also be done as a parameter override outside of the Dockerfile)
CMD [ "lambda_function.make_prediction" ]