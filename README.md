
import { CollateralTaskStepsMapperService } from '../path-to-service'; // Adjust import as needed

describe('CollateralTaskStepsMapperService - getObjValue', () => {
  let service: CollateralTaskStepsMapperService;

  beforeEach(() => {
    service = new CollateralTaskStepsMapperService();
  });

  it('should return the correct value when a valid path exists', () => {
    const docContainer = { policy: { details: { type: 'Homeowners Insurance' } } };
    const dataPath = 'policy.details.type'.split('.');
    
    const result = service.taskHydrateDataService.getObjectValue(docContainer, dataPath.join('.'));

    expect(result).toBe('Homeowners Insurance');
  });

  it('should return undefined when the path does not exist', () => {
    const docContainer = { policy: { details: {} } };
    const dataPath = 'policy.details.nonExistent'.split('.');

    const result = service.taskHydrateDataService.getObjectValue(docContainer, dataPath.join('.'));

    expect(result).toBeUndefined();
  });

  it('should return the default value when getObjValue is undefined', () => {
    const docContainer = { policy: { details: {} } };
    const dataPath = 'policy.details.type'.split('.');

    const result = service.taskHydrateDataService.getObjectValue(docContainer, dataPath.join('.')) ?? 'Wind Insurance';

    expect(result).toBe('Wind Insurance');
  });

  it('should correctly set the pathKey and assign value', () => {
    const docContainer: any = {};
    const dataMap = { name: 'policy.details.type' };
    const dataPath = dataMap.name.split('.');
    const pathKey = dataPath.pop();

    const getObjValue = service.taskHydrateDataService.getObjectValue(docContainer, dataPath.join('.')) ?? 'Wind Insurance';

    docContainer[pathKey as string] = getObjValue;

    expect(docContainer).toHaveProperty('type', 'Wind Insurance');
  });

  it('should handle an empty dataMap.name gracefully', () => {
    const docContainer = {};
    const dataMap = { name: '' };
    const dataPath = dataMap.name.split('.');
    const pathKey = dataPath.pop();

    const getObjValue = service.taskHydrateDataService.getObjectValue(docContainer, dataPath.join('.')) ?? 'Wind Insurance';

    expect(getObjValue).toBe('Wind Insurance');
    expect(pathKey).toBeUndefined();
  });

  it('should handle a single segment in dataMap.name correctly', () => {
    const docContainer = { type: 'Homeowners Insurance' };
    const dataMap = { name: 'type' };
    const dataPath = dataMap.name.split('.');
    const pathKey = dataPath.pop();

    const getObjValue = service.taskHydrateDataService.getObjectValue(docContainer, dataPath.join('.')) ?? 'Wind Insurance';

    expect(getObjValue).toBe('Homeowners Insurance');
    expect(pathKey).toBe('type');
  });

});
