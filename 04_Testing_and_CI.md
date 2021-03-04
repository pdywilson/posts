# Data Engineering Toy Project
## Testing and CI

Further, I refactored my Python code so that I can test it using `pytest`. I activated Travis CI/CD, Codecov and added a .travis.yml file to the Github repository with the following commands.
```
language: python
python:
  - "3.7"
install:
  - pip install -r requirements.txt
  - pip install -e .
  - pip install pytest-cov codecov
script:
  - pytest --cov=src tests
after_success:
  - codecov
```

Now, everytime I push a change to my project, the projects gets built on the Travis CI/CD Server and the tests are run. It also makes it possible to display the badges on the github readme file showing test coverage in % and build passing/failing.


