import React, { useEffect, useState } from 'react';
import JobAutocomplete from './JobAutocomplete';

const ParentComponent = () => {
  const [connectionOptions, setConnectionOptions] = useState([]);
  const [schemaOptions, setSchemaOptions] = useState([]);
  const [objectOptions, setObjectOptions] = useState([]);
  const [operationTagValueList, setOperationTagValueList] = useState([]);
  const [rows, setRows] = useState([
    // Initialize with your rows data structure
  ]);
  const [loadingTagValues, setLoadingTagValues] = useState({});

  useEffect(() => {
    // Assume fetchJobData is a function that fetches your jobData array
    fetchJobData().then((jobData) => {
      const newConnectionOptions = [];
      const newSchemaOptions = [];
      const newObjectOptions = [];

      jobData.forEach((job) => {
        if (job.operationTag === 'Connection Id') {
          newConnectionOptions.push({ CONNECTION_ID: job.operationTagValue, CONNECTION_NAME: job.operationTagValueGui });
        } else if (job.operationTag === 'Write Schema Name') {
          newSchemaOptions.push({ SCHEMA_NAME: job.operationTagValue, ENTITY_NAME: job.operationTagValueGui });
        } else if (job.operationTag === 'Write Table Name') {
          newObjectOptions.push({ SCHEMA_NAME: job.operationTagValue, ENTITY_NAME: job.operationTagValueGui });
        }
      });

      setConnectionOptions(newConnectionOptions);
      setSchemaOptions(newSchemaOptions);
      setObjectOptions(newObjectOptions);
    });
  }, []);

  const handleOperationTagValueChange = (id, value, selectedTagId) => {
    const updatedRows = rows.map((row) => (row.id === id ? { ...row, operationTagValue: value } : row));
    setRows(updatedRows);
  };

  const handleOperationTagValueOptionsChange = (searchValue, params) => {
    // Implement your logic to handle options change
  };

  const handleFetchSchemaObjectChange = (params) => {
    // Implement your logic to fetch schema objects
  };

  return (
    <div>
      {rows.map((row) => (
        <JobAutocomplete
          key={row.id}
          params={{ id: row.id, row }}
          rows={rows}
          operationTagValueList={operationTagValueList}
          handleOperationTagValueChange={handleOperationTagValueChange}
          handleOperationTagValueOptionsChange={handleOperationTagValueOptionsChange}
          setOperationTagValueList={setOperationTagValueList}
          loadingTagValues={loadingTagValues[row.id]}
          handleFetchSchemaObjectChange={handleFetchSchemaObjectChange}
        />
      ))}
    </div>
  );
};

export default ParentComponent;