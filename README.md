import * as React from 'react';
import { DataGrid, GridColDef, GridRenderCellParams } from '@mui/x-data-grid';
import { TextField } from '@mui/material';

type RowData = {
  id: number;
  name: string;
};

const initialRows: RowData[] = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  // Add more rows as needed
];

const DataGridWithTextField = () => {
  const [rows, setRows] = React.useState<RowData[]>(initialRows);
  const [editCellData, setEditCellData] = React.useState<{ [key: number]: string }>({});

  const handleEditCellChange = React.useCallback((id: number, value: string) => {
    setEditCellData((prev) => ({
      ...prev,
      [id]: value,
    }));
  }, []);

  const columns: GridColDef[] = React.useMemo(() => [
    { field: 'id', headerName: 'ID', width: 100 },
    {
      field: 'name',
      headerName: 'Name',
      width: 300,
      renderCell: (params: GridRenderCellParams<string>) => {
        return (
          <TextField
            value={editCellData[params.id] || params.value || ''}
            onChange={(event) => handleEditCellChange(params.id as number, event.target.value)}
            variant="outlined"
            fullWidth
          />
        );
      },
    },
  ], [editCellData, handleEditCellChange]);

  return (
    <div style={{ height: 400, width: '100%' }}>
      <DataGrid
        rows={rows}
        columns={columns}
        pageSize={5}
        rowsPerPageOptions={[5]}
      />
    </div>
  );
};

export default DataGridWithTextField;