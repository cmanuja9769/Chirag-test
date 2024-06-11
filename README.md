import React from 'react';
import { useDagContext } from '../DagContext/index';
import { Box, Typography, Grid } from '@mui/material';

const DagPreview: React.FC = () => {
  const dagContext = useDagContext();
  const { dagData, headers } = dagContext;

  const headersData = headers.filter((header) => header.SEQUENCE <= 3);

  return (
    <Box className="tw-mt-5 tw-p-4 tw-border-solid tw-border-2 tw-border-gray-300">
      <Typography variant="h6" component="h6" className="tw-bg-gray-200 tw-text-black tw-font-bold tw-p-2">
        DAG Preview
      </Typography>
      <Grid container spacing={3} direction="row" alignItems="flex-start" justifyContent="flex-start">
        {headersData.map((header) => (
          <Grid item xs={12} sm={6} key={header.name}>
            <Box className="tw-mb-4">
              <Typography variant="body1" className="tw-font-medium">
                {header.label}:
              </Typography>
              <Typography variant="body2" className="tw-text-gray-600">
                {dagData[header.name]}
              </Typography>
            </Box>
          </Grid>
        ))}
      </Grid>
    </Box>
  );
};

export default DagPreview;