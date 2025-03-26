# future-shock-fps


# Check whether database is up and running
if [ $status -eq 0 ]; then
  echo "#########################"
  echo "DATABASE IS READY TO USE!"
  echo "#########################"

  # Check if ORACL PDB exists
  echo "Checking if ORACL PDB exists..."
  PDB_EXISTS=$(
    sqlplus -S / as sysdba <<EOF
    SET HEADING OFF;
    SET FEEDBACK OFF;
    SELECT COUNT(*) FROM DBA_PDBS WHERE PDB_NAME = 'ORACL';
    EXIT;
EOF
  )

  if [ "$PDB_EXISTS" -eq 0 ]; then
    echo "Creating ORACL PDB..."

    # Pass the password as an argument to the SQL script
    sqlplus / as sysdba <<EOF
    @/opt/oracle/scripts/setup/create_oracl_pdb.sql $ORACLE_PWD
    EXIT;
EOF

    echo "ORACL PDB created successfully!"
  else
    echo "ORACL PDB already exists."
  fi

  # Execute startup script for extensions
  "$ORACLE_BASE"/"$USER_SCRIPTS_FILE" "$ORACLE_BASE"/scripts/extensions/startup

  # Execute custom provided startup scripts
  "$ORACLE_BASE"/"$USER_SCRIPTS_FILE" "$ORACLE_BASE"/scripts/startup

else
  echo "#####################################"
  echo "########### E R R O R ###############"
  echo "DATABASE SETUP WAS NOT SUCCESSFUL!"
  echo "Please check output for further info!"
  echo "########### E R R O R ###############"
  echo "#####################################"
fi
