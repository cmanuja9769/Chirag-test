export const ConnectionEndpoints = Object.freeze({
  getConnections: '/connections',
  getInitialFields: '/getFormFields',
  postConnection: '/connections',
  searchConnections: '/criteria?identifier=IDENTIFIER_PTCDH_SNOWFLAKE_CONNECTIONS_WITH_QUERYSTRING',
  getSchemaByConnection: '/schema/getSchemaByConnection'
});

export const DagEndpoints = Object.freeze({
  getDag: '/dag'
});

export const JobEndpoints = Object.freeze({
  getJobType: '/criteria?identifier=IDENTIFIER_JOB_TYPE'
});


import axios from 'axios';
import { ConnectionEndpoints } from './apiendpoints';

// Your base URL
const BASE_URL = 'http://localhost:3001/api/v1';

const fetchConnections = async (searchValue?: string) => {
  try {
    const response = await axios.get(
      `${BASE_URL}${ConnectionEndpoints.searchConnections}&search=${searchValue}`
    );
    setConnectionOptions(response?.data);
  } catch (error) {
    handleToast(t('fetch_error'), TOAST_TYPE.ERROR);
  }
};

const fetchSchemas = async (connectionId?: string) => {
  try {
    const response = await axios.get(
      `${BASE_URL}${ConnectionEndpoints.getSchemaByConnection}?connectionId=${connectionId}`
    );
    setSchemaOptions(response?.data?.schema);
  } catch (error) {
    handleToast(t('fetch_error'), TOAST_TYPE.ERROR);
  }
};

const fetchObjects = async (connectionId?: string, schemaName?: string) => {
  try {
    const response = await axios.get(
      `${BASE_URL}${ConnectionEndpoints.getSchemaByConnection}?connectionId=${connectionId}&schemaName=${schemaName}`
    );
    setObjectOptions(response?.data?.objects);
  } catch (error) {
    handleToast(t('fetch_error'), TOAST_TYPE.ERROR);
  }
};