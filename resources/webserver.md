# start backend
cd backend
<!-- go build -o server && ./server -local -config config.json -->
# Path of binaries, dbs, jobs -> config.json
go build -o ../resources/server && ../resources/server -local -config config.json

# start frontend
FRONTEND_APP=foldseek npm run frontend:dev

# add server backend temporarily to main.js
apiBase = "http://127.0.0.1:8081"

SAFE=exdb
DB=exdb
VERSION=1
TAXONOMY=false
echo "{\n  \"status\": \"COMPLETE\",\n  \"name\": \"${DB}\",\n  \"version\": \"${VERSION}\",\n  \"path\": \"${SAFE}\",\n  \"taxonomy\": ${TAXONOMY},\n  \"motif\": true,\n   \"default\": true,\n  \"order\": 0,\n  \"index\": \"\",\n  \"search\": \"\"\n}" > "${SAFE}.params"

ex pdb
```
ATOM      1  N   GLY A   1       1.312  33.477  23.992  1.00 43.60      A    N  
ATOM      2  CA  GLY A   1       2.265  34.188  24.838  1.00 45.83      A    C  
ATOM      3  C   GLY A   1       3.423  33.244  25.186  1.00 43.55      A    C  
ATOM      4  O   GLY A   1       3.137  32.046  25.235  1.00 43.92      A    O  
ATOM      5  N   LEU A   2       4.627  33.736  25.542  1.00 40.44      A    N  
ATOM      6  CA  LEU A   2       5.831  32.906  25.698  1.00 40.29      A    C  
ATOM      7  C   LEU A   2       6.267  32.248  26.994  1.00 40.05      A    C  
ATOM      8  O   LEU A   2       6.470  32.916  28.012  1.00 46.19      A    O  
ATOM      9  CB  LEU A   2       7.082  33.667  25.227  1.00 40.02      A    C  
ATOM     10  CG  LEU A   2       7.541  33.591  23.780  1.00 39.23      A    C  
ATOM     11  CD1 LEU A   2       6.360  33.891  22.882  1.00 43.77      A    C  
ATOM     12  CD2 LEU A   2       8.646  34.598  23.514  1.00 40.34      A    C  
ATOM     13  N   THR A   3       6.492  30.942  26.980  1.00 37.08      A    N  
ATOM     14  CA  THR A   3       6.989  30.243  28.144  1.00 37.31      A    C  
ATOM     15  C   THR A   3       8.469  29.939  27.932  1.00 34.00      A    C  
ATOM     16  O   THR A   3       8.975  29.951  26.808  1.00 37.25      A    O  
ATOM     17  CB  THR A   3       6.116  28.952  28.333  1.00 42.03      A    C  
ATOM     18  OG1 THR A   3       6.172  28.181  27.127  1.00 44.60      A    O  
ATOM     19  CG2 THR A   3       4.665  29.291  28.671  1.00 40.40      A    C  
```