// context.tsx
import React, { createContext, useState, useContext, ReactNode } from 'react';

interface AppState {
  selectedOption: string;
  checkboxStates: { [key: string]: boolean };
  previewData: { [key: string]: string };
}

interface AppContextProps {
  state: AppState;
  setState: React.Dispatch<React.SetStateAction<AppState>>;
}

const AppContext = createContext<AppContextProps | undefined>(undefined);

export const AppProvider = ({ children }: { children: ReactNode }) => {
  const [state, setState] = useState<AppState>({
    selectedOption: '',
    checkboxStates: {},
    previewData: {},
  });

  return (
    <AppContext.Provider value={{ state, setState }}>
      {children}
    </AppContext.Provider>
  );
};

export const useAppContext = () => {
  const context = useContext(AppContext);
  if (!context) {
    throw new Error('useAppContext must be used within an AppProvider');
  }
  return context;
};


// AccordionComponent.tsx
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
} from '@mui/material';
import ExpandMoreIcon from '@mui/icons-material/ExpandMore';
import { useAppContext } from './context';

const options = [
  { value: 'Merge', checkboxes: ['Join', 'Delta Detection'] },
  { value: 'Truncate&Load', checkboxes: ['Join', 'Filter', 'Group By', 'Distinct'] },
  { value: 'Append', checkboxes: ['Join', 'Filter', 'Group By'] },
  { value: 'Update', checkboxes: ['Distinct', 'Delta Detection'] },
];

const AccordionComponent: React.FC = () => {
  const { state, setState } = useAppContext();
  const [openModal, setOpenModal] = useState(false);
  const [currentCheckbox, setCurrentCheckbox] = useState('');

  const handleSelectChange = (event: React.ChangeEvent<{ value: unknown }>) => {
    const selectedOption = event.target.value as string;
    setState(prevState => ({
      ...prevState,
      selectedOption,
      checkboxStates: {},
    }));
  };

  const handleCheckboxChange = (checkbox: string) => {
    setState(prevState => ({
      ...prevState,
      checkboxStates: {
        ...prevState.checkboxStates,
        [checkbox]: !prevState.checkboxStates[checkbox],
      },
    }));
    setCurrentCheckbox(checkbox);
    setOpenModal(true);
  };

  const handleModalSave = () => {
    setOpenModal(false);
  };

  const renderCheckboxes = () => {
    const selectedOption = options.find(option => option.value === state.selectedOption);
    return selectedOption?.checkboxes.map(checkbox => (
      <FormControlLabel
        key={checkbox}
        control={
          <Checkbox
            checked={state.checkboxStates[checkbox] || false}
            onChange={() => handleCheckboxChange(checkbox)}
          />
        }
        label={checkbox}
      />
    ));
  };

  const allCheckboxesSelected = Object.values(state.checkboxStates).some(value => value);

  return (
    <div>
      <Accordion defaultExpanded={false}>
        <AccordionSummary expandIcon={<ExpandMoreIcon />}>
          <Typography>Configuration</Typography>
        </AccordionSummary>
        <AccordionDetails>
          <FormControl fullWidth>
            <InputLabel>Select Option</InputLabel>
            <Select value={state.selectedOption} onChange={handleSelectChange}>
              {options.map(option => (
                <MenuItem key={option.value} value={option.value}>
                  {option.value}
                </MenuItem>
              ))}
            </Select>
          </FormControl>
          <div>
            {renderCheckboxes()}
          </div>
          <Button variant="contained" disabled={!allCheckboxesSelected}>
            Preview Configuration
          </Button>
        </AccordionDetails>
      </Accordion>

      <Modal open={openModal} onClose={() => setOpenModal(false)}>
        <div style={{ padding: '20px', backgroundColor: 'white', margin: '20px auto', width: '300px' }}>
          <TextField
            label="Configuration"
            fullWidth
            value={state.previewData[currentCheckbox] || ''}
            onChange={(e) =>
              setState(prevState => ({
                ...prevState,
                previewData: {
                  ...prevState.previewData,
                  [currentCheckbox]: e.target.value,
                },
              }))
            }
          />
          <Button onClick={handleModalSave}>Save</Button>
        </div>
      </Modal>
    </div>
  );
};

export default AccordionComponent;
