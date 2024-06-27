import React, { useState } from 'react';
import {
  Accordion,
  AccordionSummary,
  AccordionDetails,
  Typography,
  Select,
  MenuItem,
  FormControl,
  InputLabel,
  Checkbox,
  FormControlLabel,
  Button,
  Modal,
  TextField,
  SelectChangeEvent,
  Box,
  Grid
} from '@mui/material';
import ExpandMoreIcon from '@mui/icons-material/ExpandMore';
import { useJobContext } from '../../../../context/jobContext';

const options = [
  { value: 'Merge', checkboxes: ['Join', 'Delta Detection'] },
  { value: 'Truncate&Load', checkboxes: ['Join', 'Filter', 'Group By', 'Distinct'] },
  { value: 'Append', checkboxes: ['Join', 'Filter', 'Group By'] },
  { value: 'Update', checkboxes: ['Distinct', 'Delta Detection'] }
];

const DatasetConfiguration: React.FC = () => {
  const { jobData, updateJobData } = useJobContext();
  const [openModal, setOpenModal] = useState(false);
  const [currentCheckbox, setCurrentCheckbox] = useState('');

  const handleSelectChange = (event: SelectChangeEvent) => {
    const selectedOption = event.target.value as string;
    updateJobData((prevState) => ({
      ...prevState,
      selectedOption,
      checkboxStates: {}
    }));
  };

  const handleCheckboxChange = (checkbox: string) => {
    updateJobData((prevState) => ({
      ...prevState,
      checkboxStates: {
        ...prevState.checkboxStates,
        [checkbox]: !prevState.checkboxStates[checkbox]
      }
    }));
    setCurrentCheckbox(checkbox);
    setOpenModal(true);
  };

  const handleModalSave = () => {
    setOpenModal(false);
  };

  const renderCheckboxes = () => {
    const selectedOption = options.find((option) => option.value === jobData.selectedOption);
    return selectedOption?.checkboxes.map((checkbox) => (
      <>
        <Grid container spacing={4} direction="row" alignItems="center" justifyContent="center">
          <Grid item xs={12} sm={12} display="inline-flex" alignItems="center">
            <Grid item xs={12} sm={4} marginTop="1vh" display="inline-flex">
              <FormControlLabel
                key={checkbox}
                control={<Checkbox checked={jobData.checkboxStates[checkbox] || false} onChange={() => handleCheckboxChange(checkbox)} />}
                label={checkbox}
              />
            </Grid>
            <Grid item xs={12} sm={8} marginTop="1vh">
              <Button variant="contained" disabled={!allCheckboxesSelected}>
                Preview Configuration
              </Button>
            </Grid>
          </Grid>
        </Grid>
      </>
    ));
  };

  const allCheckboxesSelected = Object.values(jobData.checkboxStates).some((value) => value);

  return (
    <Box className="tw-mt-5 tw-block tw-pb-2">
      <Accordion defaultExpanded={false} className="tw-border-solid tw-border-2 tw-border-pfizerBlue">
        <AccordionSummary
          expandIcon={<ExpandMoreIcon className="tw-text-white tw-pe-2 tw-text-[2vw]" />}
          className="tw-bg-pfizerBlue tw-text-white tw-font-bold tw-ps-1"
        >
          <Typography variant="h6" component="h6" paddingLeft={1}>
            Dataset Level Configutation
          </Typography>
        </AccordionSummary>
        <Box display="inline-flex" alignItems="center">
          <AccordionDetails className="tw-pt-5 tw-ps-1">
            <Grid container spacing={2} direction="row" alignItems="center" justifyContent="center">
              <Grid item xs={12} sm={12} display="inline-flex" justifyContent="space-around" alignItems="center">
                <Grid item xs={12} sm={2} marginTop="1vh" display="inline-flex">
                  <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>Operation</Typography>
                </Grid>
                <Grid item xs={12} sm={6}>
                  <FormControl fullWidth>
                    <Select value={jobData.selectedOption} onChange={handleSelectChange} placeholder="Select Option">
                      {options.map((option) => (
                        <MenuItem key={option.value} value={option.value}>
                          {option.value}
                        </MenuItem>
                      ))}
                    </Select>
                  </FormControl>
                </Grid>
              </Grid>
            </Grid>
            <Box alignItems="center" className="tw-ps-10 tw-mt-5">
              {renderCheckboxes()}
            </Box>
          </AccordionDetails>
        </Box>
      </Accordion>

      <Modal open={openModal} onClose={() => setOpenModal(false)}>
        <div style={{ padding: '20px', backgroundColor: 'white', margin: '20px auto', width: '300px' }}>
          <TextField
            label="Configuration"
            fullWidth
            value={jobData.previewData[currentCheckbox] || ''}
            onChange={(e) =>
              updateJobData((prevState) => ({
                ...prevState,
                previewData: {
                  ...prevState.previewData,
                  [currentCheckbox]: e.target.value
                }
              }))
            }
          />
          <Button onClick={handleModalSave}>Save</Button>
        </div>
      </Modal>
    </Box>
  );
};

export default DatasetConfiguration;
