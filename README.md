import React, { useState } from 'react';
import {
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  TablePagination,
  Button,
} from '@mui/material';

interface Data {
  id: number;
  name: string;
}

const createData = (id: number, name: string): Data => {
  return { id, name };
};

const MyTable: React.FC = () => {
  const [rows, setRows] = useState<Data[]>([
    createData(1, 'Row 1'),
    createData(2, 'Row 2'),
    createData(3, 'Row 3'),
    createData(4, 'Row 4'),
    createData(5, 'Row 5'),
    createData(6, 'Row 6'),
    createData(7, 'Row 7'),
    createData(8, 'Row 8'),
    createData(9, 'Row 9'),
    createData(10, 'Row 10'),
  ]);
  const [page, setPage] = useState(0);
  const [rowsPerPage] = useState(10);

  const handleChangePage = (event: unknown, newPage: number) => {
    setPage(newPage);
  };

  const handleAddRow = () => {
    const newRow = createData(rows.length + 1, `Row ${rows.length + 1}`);
    setRows([...rows, newRow]);
    if ((page + 1) * rowsPerPage <= rows.length) {
      setPage(page + 1);
    }
  };

  return (
    <Paper>
      <TableContainer>
        <Table>
          <TableHead>
            <TableRow>
              <TableCell>ID</TableCell>
              <TableCell>Name</TableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {(rowsPerPage > 0
              ? rows.slice(page * rowsPerPage, page * rowsPerPage + rowsPerPage)
              : rows
            ).map((row) => (
              <TableRow key={row.id}>
                <TableCell>{row.id}</TableCell>
                <TableCell>{row.name}</TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </TableContainer>
      <TablePagination
        component="div"
        count={rows.length}
        rowsPerPage={rowsPerPage}
        page={page}
        onPageChange={handleChangePage}
        rowsPerPageOptions={[10]}
      />
      <Button variant="contained" color="primary" onClick={handleAddRow}>
        Add Row
      </Button>
    </Paper>
  );
};

export default MyTable;