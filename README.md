import React, { useState, useCallback, useMemo } from 'react';
import { DataGrid } from '@mui/x-data-grid';
import TextField from '@mui/material/TextField';
import _ from 'lodash';

const MyDataGrid = () => {
  const [rows, setRows] = useState([
    { id: 1, name: '' },
    { id: 2, name: '' },
    // Add more rows as needed
  ]);

  const handleChange = useCallback((id, value) => {
    setRows((prevRows) =>
      prevRows.map((row) => (row.id === id ? { ...row, name: value } : row))
    );
  }, []);

  const debouncedHandleChange = useMemo(
    () => _.debounce(handleChange, 300),
    [handleChange]
  );

  const columns = [
    {
      field: 'name',
      headerName: 'Name',
      width: 150,
      renderCell: (params) => (
        <MemoizedTextField
          value={params.row.name}
          onChange={(e) => debouncedHandleChange(params.row.id, e.target.value)}
        />
      ),
    },
    // Add more columns as needed
  ];

  return <DataGrid rows={rows} columns={columns} />;
};

const MemoizedTextField = React.memo(({ value, onChange }) => {
  return <TextField value={value} onChange={onChange} />;
});

export default MyDataGrid;