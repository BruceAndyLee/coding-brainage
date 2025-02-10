<%*
	const DEFAULT_PROJECT_NAME = "LOL_NEW_PROJECT";
	const planner_kanban_note_name = "planner";
	
	const PROJECT_FOLDER = `Coding brainage/${DEFAULT_PROJECT_NAME}/`;
	const readme_tfile = tp.file.find_tfile("project-readme-template");
	const planner_kanban_tfile = tp.file.find_tfile("project-planner-template");
	
	await this.app.vault.createFolder(PROJECT_FOLDER);
	await tp.file.move(PROJECT_FOLDER + "README");
	tR += await tp.file.include(readme_tfile);
	
	await tp.file.create_new(planner_kanban_tfile, planner_kanban_note_name, false, PROJECT_FOLDER);
%>