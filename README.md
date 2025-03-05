describe('mapDocData', () => {
  it('should process DOCList type correctly', () => {
    const docContainer = { dataMapModel: [{ type: 'DOCLIST', name: 'testName' }] };
    spyOn(service.taskHydrateDataService, 'getObjectValue').and.returnValue({ test: 'value' });

    service.mapDocData(docContainer);

    expect(service.taskHydrateDataService.getObjectValue).toHaveBeenCalled();
  });

  it('should process DOCDESC type correctly', () => {
    const docContainer = { dataMapModel: [{ type: 'DOCDESC', name: 'path.to.value' }] };
    spyOn(service.taskHydrateDataService, 'getObjectValue').and.returnValue('description');

    service.mapDocData(docContainer);

    expect(service.taskHydrateDataService.getObjectValue).toHaveBeenCalled();
  });

  it('should process NSHDATA type correctly', () => {
    const docContainer = { dataMapModel: [{ type: 'NSHDATA', name: 'testNSH' }] };
    spyOn(service.taskHydrateDataService, 'getObjectValue').and.returnValue(null);

    service.mapDocData(docContainer);

    expect(docContainer.notesAndStipulationHistory['notesAndStipHistoryData']).toEqual([]);
  });

  it('should not process when dataMapModel is empty', () => {
    const docContainer = { dataMapModel: [] };
    spyOn(service.taskHydrateDataService, 'getObjectValue');

    service.mapDocData(docContainer);

    expect(service.taskHydrateDataService.getObjectValue).not.toHaveBeenCalled();
  });
});
