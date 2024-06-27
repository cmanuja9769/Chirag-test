import { createContext, useContext, useState, ReactNode } from 'react';
import { JobContextProps, JobProviderProps } from '../components/Jobs/job.interface';

const JobContext = createContext<JobContextProps>({
  jobData: null,
  step: 0,
  headers: [],
  charCount: 0,
  updateJobData: () => {},
  updateStage: () => {},
  updateHeaders: () => {},
  setCharCount: () => {},
  validateJobDetails: () => Promise.resolve(),
  validateJobSource: () => Promise.resolve(),
  validateJobTarget: () => Promise.resolve(),
  setJobDetailsValidation: () => {},
  setJobSourceValidation: () => {},
  setJobTargetValidation: () => {},
  isNexButtonClicked: false,
  handleNextClick: () => {}
});

export const JobProvider = ({ children }: JobProviderProps) => {
  const [jobData, setJobData] = useState({
    job_name: '',
    description: '',
    job_type: '',
    source_conName: '',
    source_schema: '',
    source_object: '',
    target_conName: '',
    target_object: '',
    target_schema: '',
    selectedOption: '',
    checkboxStates: {},
    previewData: {}
  });

  const [headers, setHeaders] = useState([]);
  const [step, setStep] = useState(0);
  const [charCount, setCharCount] = useState(256);
  const [validateJobDetails, setJobDetailsValidation] = useState(() => () => Promise.resolve());
  const [validateJobSource, setJobSourceValidation] = useState(() => () => Promise.resolve());
  const [validateJobTarget, setJobTargetValidation] = useState(() => () => Promise.resolve());
  const [isNexButtonClicked, setNextButtonClicked] = useState(false);

  const updateJobData = (value: any) => {
    setJobData(value);
  };

  const updateStage = (value: any) => {
    setStep(value);
  };

  const updateHeaders = (value: any) => {
    setHeaders(value);
  };

  const handleNextClick = (value: boolean) => {
    setNextButtonClicked(value);
  };

  const updateCheckboxState = (checkbox: string) => {
    setJobData((prevState) => ({
      ...prevState,
      checkboxStates: {
        ...prevState.checkboxStates,
        [checkbox]: !prevState.checkboxStates[checkbox],
      },
    }));
  };

  const updateSelectedOption = (option: string) => {
    setJobData((prevState) => ({
      ...prevState,
      selectedOption: option,
      checkboxStates: {},
    }));
  };

  const updatePreviewData = (checkbox: string, data: string) => {
    setJobData((prevState) => ({
      ...prevState,
      previewData: {
        ...prevState.previewData,
        [checkbox]: data,
      },
    }));
  };

  const context: JobContextProps = {
    jobData,
    step,
    headers,
    charCount,
    updateJobData,
    updateStage,
    updateHeaders,
    setCharCount,
    validateJobDetails,
    validateJobSource,
    validateJobTarget,
    setJobDetailsValidation,
    setJobSourceValidation,
    setJobTargetValidation,
    isNexButtonClicked,
    handleNextClick,
    updateCheckboxState,
    updateSelectedOption,
    updatePreviewData,
  };

  return <JobContext.Provider value={context}>{children}</JobContext.Provider>;
};

export const useJobContext = () => useContext(JobContext);

import { ReactNode } from 'react';

export interface JobDataProps {
  id?: number;
  name?: string;
  tags?: string[];
  createdtime?: string;
  description?: string;
  status?: string;
}

export interface JobData {
  job_name?: string;
  description?: string;
  job_type?: string;
  source_conName?: string;
  source_schema?: string;
  source_object?: string;
  target_conName?: string;
  target_object?: string;
  target_schema?: string;
  selectedOption?: string;
  checkboxStates?: { [key: string]: boolean };
  previewData?: { [key: string]: string };
}

export interface JobContextProps {
  jobData?: JobData;
  step?: number;
  headers: any[];
  charCount: number;
  updateJobData?: React.Dispatch<React.SetStateAction<JobData>>;
  updateStage?: React.Dispatch<React.SetStateAction<number>>;
  setValidateJobDetailsForm?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  setValidateJobParamsForm?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  updateHeaders?: React.Dispatch<React.SetStateAction<any[]>>;
  setCharCount: React.Dispatch<React.SetStateAction<number>>;
  validateJobDetails?: () => Promise<void>;
  validateJobSource?: () => Promise<void>;
  validateJobTarget?: () => Promise<void>;
  setJobDetailsValidation?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  setJobSourceValidation?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  setJobTargetValidation?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  isNexButtonClicked?: boolean;
  handleNextClick?: React.Dispatch<React.SetStateAction<boolean>>;
  updateCheckboxState?: (checkbox: string) => void;
  updateSelectedOption?: (option: string) => void;
  updatePreviewData?: (checkbox: string, data: string) => void;
}

export interface JobProviderProps {
  children?: ReactNode;
}

export interface JobHeaderProps {
  onClose?: () => void;
  step?: number;
  onBack?: () => void;
}




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
  const [previewModalOpen, setPreviewModalOpen] = useState(false);
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

  const handlePreviewButtonClick = () => {
    setPreviewModalOpen(true);
  };

  const renderCheckboxes = () => {
    const selectedOption = options.find((option) => option.value === jobData.selectedOption);
    return selectedOption?.checkboxes.map((checkbox) => (
      <Grid container spacing={4} direction="row" alignItems="center" justifyContent="center" key={checkbox}>
        <Grid item xs={12} sm={12} display="inline-flex" alignItems="center">
          <Grid item xs={12} sm={4} marginTop="1vh" display="inline-flex">
            <FormControlLabel
              control={<Checkbox checked={jobData.checkboxStates[checkbox] || false} onChange={() => handleCheckboxChange(checkbox)} />}
              label={checkbox}
            />
          </Grid>
          <Grid item xs={12} sm={8} marginTop="1vh">
            <Button variant="contained" disabled={!jobData.checkboxStates[checkbox]} onClick={handlePreviewButtonClick}>
              Preview Configuration
            </Button>
          </Grid>
        </Grid>
      </Grid>
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
            Dataset Level Configuration
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
        <Box sx={{ padding: '20px', backgroundColor: 'white', margin: '20px auto', width: '300px' }}>
          <Typography variant="h6">{currentCheckbox} Configuration</Typography>
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
          <Button onClick={() => setOpenModal(false)}>Cancel</Button>
        </Box>
      </Modal>

      <Modal open={previewModalOpen} onClose={() => setPreviewModalOpen(false)}>
        <Box sx={{ padding: '20px', backgroundColor: 'white', margin: '20px auto', width: '300px' }}>
          <Typography variant="h6">Preview Configuration</Typography>
          <Typography>{jobData.previewData[currentCheckbox] || 'No data available'}</Typography>
          <Button onClick={() => setPreviewModalOpen(false)}>Close</Button>
        </Box>
      </Modal>
    </Box>
  );
};

export default DatasetConfiguration;