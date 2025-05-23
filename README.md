import { useState, useEffect } from 'react';
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Select, SelectTrigger, SelectValue, SelectContent, SelectItem } from "@/components/ui/select";
import { Tabs, TabsList, TabsTrigger, TabsContent } from "@/components/ui/tabs";
import { Table, TableHeader, TableRow, TableHead, TableBody, TableCell } from "@/components/ui/table";
import { Calendar } from "@/components/ui/calendar";
import { Progress } from "@/components/ui/progress";
import { Upload, GanttChart, Clock, HardHat, Users, DollarSign, AlertTriangle, CheckCircle } from 'lucide-react';
import { DragDropContext, Droppable, Draggable } from 'react-beautiful-dnd';

// Sample initial data
const initialProjects = [
  {
    id: 1,
    name: "Downtown Highrise",
    description: "45-story residential tower with retail space",
    status: "active",
    startDate: new Date(2023, 5, 1),
    endDate: new Date(2025, 11, 30),
    budget: 85000000,
    spent: 32000000,
    location: "123 Main St, Metropolis",
    team: ["John Smith", "Maria Garcia", "Lee Chen"],
    phases: ["Excavation", "Foundation", "Structure", "Enclosure", "Interiors"]
  },
  {
    id: 2,
    name: "Riverside Hospital Wing",
    description: "New emergency department expansion",
    status: "planning",
    startDate: new Date(2024, 2, 15),
    endDate: new Date(2026, 8, 1),
    budget: 120000000,
    spent: 5000000,
    location: "456 Health Ave, Riverside",
    team: ["Sarah Johnson", "David Kim"],
    phases: ["Design", "Permitting", "Construction"]
  }
];

const initialTasks = [
  {
    id: 1,
    projectId: 1,
    name: "Site excavation",
    description: "Excavate to 30ft depth for foundation",
    status: "completed",
    phase: "Excavation",
    assignedTo: "John Smith",
    startDate: new Date(2023, 5, 1),
    endDate: new Date(2023, 6, 15),
    dependencies: [],
    progress: 100
  },
  {
    id: 2,
    projectId: 1,
    name: "Foundation pouring",
    description: "Pour reinforced concrete foundation",
    status: "in-progress",
    phase: "Foundation",
    assignedTo: "Maria Garcia",
    startDate: new Date(2023, 6, 16),
    endDate: new Date(2023, 8, 1),
    dependencies: [1],
    progress: 65
  },
  {
    id: 3,
    projectId: 2,
    name: "Architectural design",
    description: "Complete all architectural drawings",
    status: "not-started",
    phase: "Design",
    assignedTo: "Sarah Johnson",
    startDate: new Date(2024, 2, 15),
    endDate: new Date(2024, 5, 30),
    dependencies: [],
    progress: 0
  }
];

export default function ConstructionPMApp() {
  const [projects, setProjects] = useState(initialProjects);
  const [tasks, setTasks] = useState(initialTasks);
  const [newProject, setNewProject] = useState({
    name: "",
    description: "",
    status: "planning",
    startDate: new Date(),
    endDate: new Date(),
    budget: 0,
    location: ""
  });
  const [newTask, setNewTask] = useState({
    name: "",
    description: "",
    projectId: "",
    status: "not-started",
    phase: "",
    assignedTo: "",
    startDate: new Date(),
    endDate: new Date(),
    dependencies: [],
    progress: 0
  });
  const [selectedProject, setSelectedProject] = useState(null);
  const [activeTab, setActiveTab] = useState("dashboard");
  const [date, setDate] = useState(new Date());
  const [documents, setDocuments] = useState([]);
  const [teamMembers, setTeamMembers] = useState([
    "John Smith", "Maria Garcia", "Lee Chen", "Sarah Johnson", "David Kim", "Alex Wong"
  ]);

  // Calculate project progress
  const calculateProjectProgress = (projectId) => {
    const projectTasks = tasks.filter(task => task.projectId === projectId);
    if (projectTasks.length === 0) return 0;
    const totalProgress = projectTasks.reduce((sum, task) => sum + task.progress, 0);
    return Math.round(totalProgress / projectTasks.length);
  };

  // Add project with full details
  const addProject = () => {
    if (newProject.name) {
      const project = {
        ...newProject,
        id: Date.now(),
        spent: 0,
        team: [],
        phases: []
      };
      setProjects([...projects, project]);
      setNewProject({
        name: "",
        description: "",
        status: "planning",
        startDate: new Date(),
        endDate: new Date(),
        budget: 0,
        location: ""
      });
    }
  };

  // Add task with dependencies
  const addTask = () => {
    if (newTask.name && newTask.projectId) {
      const task = {
        ...newTask,
        id: Date.now()
      };
      setTasks([...tasks, task]);
      setNewTask({
        name: "",
        description: "",
        projectId: "",
        status: "not-started",
        phase: "",
        assignedTo: "",
        startDate: new Date(),
        endDate: new Date(),
        dependencies: [],
        progress: 0
      });
    }
  };

  // Handle file upload
  const handleFileUpload = (e) => {
    const files = Array.from(e.target.files).map(file => ({
      id: Date.now() + Math.random(),
      name: file.name,
      type: file.type,
      size: file.size,
      uploaded: new Date(),
      projectId: selectedProject?.id || null
    }));
    setDocuments([...documents, ...files]);
  };

  // Handle task status change
  const updateTaskStatus = (taskId, status) => {
    setTasks(tasks.map(task => 
      task.id === taskId ? { ...task, status, progress: status === 'completed' ? 100 : task.progress } : task
    ));
  };

  // Handle drag and drop for tasks
  const onDragEnd = (result) => {
    if (!result.destination) return;
    
    const items = Array.from(tasks);
    const [reorderedItem] = items.splice(result.source.index, 1);
    items.splice(result.destination.index, 0, reorderedItem);
    
    setTasks(items);
  };

  // Get tasks for selected project
  const getProjectTasks = (projectId) => {
    return tasks.filter(task => task.projectId === projectId);
  };

  // Get critical path (simplified)
  const getCriticalPath = (projectId) => {
    const projectTasks = getProjectTasks(projectId);
    return projectTasks.filter(task => 
      task.dependencies.length > 0 || 
      projectTasks.some(t => t.dependencies.includes(task.id))
      .sort((a, b) => a.endDate - b.endDate);
  };

  return (
    <div className="p-6 min-h-screen bg-gray-50">
      <div className="max-w-7xl mx-auto">
        <header className="flex justify-between items-center mb-8">
          <div className="flex items-center gap-4">
            <HardHat className="h-8 w-8 text-orange-500" />
            <h1 className="text-3xl font-bold">BuildTrack Pro</h1>
          </div>
          <div className="flex items-center gap-2">
            <Clock className="h-5 w-5 text-gray-500" />
            <span>{date.toLocaleDateString()}</span>
          </div>
        </header>

        <Tabs value={activeTab} onValueChange={setActiveTab}>
          <TabsList className="grid w-full grid-cols-5">
            <TabsTrigger value="dashboard" className="flex gap-2 items-center">
              <GanttChart className="h-4 w-4" /> Dashboard
            </TabsTrigger>
            <TabsTrigger value="projects" className="flex gap-2 items-center">
              <HardHat className="h-4 w-4" /> Projects
            </TabsTrigger>
            <TabsTrigger value="tasks" className="flex gap-2 items-center">
              <CheckCircle className="h-4 w-4" /> Tasks
            </TabsTrigger>
            <TabsTrigger value="documents" className="flex gap-2 items-center">
              <Upload className="h-4 w-4" /> Documents
            </TabsTrigger>
            <TabsTrigger value="team" className="flex gap-2 items-center">
              <Users className="h-4 w-4" /> Team
            </TabsTrigger>
          </TabsList>

          <TabsContent value="dashboard" className="mt-6">
            <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
              <Card>
                <CardHeader>
                  <CardTitle className="flex justify-between">
                    <span>Active Projects</span>
                    <span className="text-green-600">{projects.filter(p => p.status === 'active').length}</span>
                  </CardTitle>
                </CardHeader>
                <CardContent>
                  {projects.filter(p => p.status === 'active').map(project => (
                    <div key={project.id} className="mb-4">
                      <div className="flex justify-between mb-1">
                        <span className="font-medium">{project.name}</span>
                        <span>{calculateProjectProgress(project.id)}%</span>
                      </div>
                      <Progress value={calculateProjectProgress(project.id)} className="h-2" />
                    </div>
                  ))}
                </CardContent>
              </Card>

              <Card>
                <CardHeader>
                  <CardTitle className="flex justify-between">
                    <span>Upcoming Deadlines</span>
                    <AlertTriangle className="h-5 w-5 text-yellow-500" />
                  </CardTitle>
                </CardHeader>
                <CardContent>
                  <Table>
                    <TableBody>
                      {tasks
                        .filter(task => task.status !== 'completed')
                        .sort((a, b) => a.endDate - b.endDate)
                        .slice(0, 5)
                        .map(task => (
                          <TableRow key={task.id}>
                            <TableCell className="font-medium">{task.name}</TableCell>
                            <TableCell>{projects.find(p => p.id === task.projectId)?.name}</TableCell>
                            <TableCell>{task.endDate.toLocaleDateString()}</TableCell>
                          </TableRow>
                        ))}
                    </TableBody>
                  </Table>
                </CardContent>
              </Card>

              <Card>
                <CardHeader>
                  <CardTitle className="flex justify-between">
                    <span>Budget Overview</span>
                    <DollarSign className="h-5 w-5 text-green-500" />
                  </CardTitle>
                </CardHeader>
                <CardContent>
                  {projects.map(project => (
                    <div key={project.id} className="mb-4">
                      <div className="flex justify-between mb-1">
                        <span className="font-medium">{project.name}</span>
                        <span>${(project.spent / 1000000).toFixed(1)}M / ${(project.budget / 1000000).toFixed(1)}M</span>
                      </div>
                      <Progress 
                        value={(project.spent / project.budget) * 100} 
                        className="h-2" 
                        indicatorColor={project.spent > project.budget ? 'bg-red-500' : 'bg-green-500'}
                      />
                    </div>
                  ))}
                </CardContent>
              </Card>
            </div>

            <Card className="mt-6">
              <CardHeader>
                <CardTitle>Project Timeline</CardTitle>
                <CardDescription>Gantt-style overview of all projects</CardDescription>
              </CardHeader>
              <CardContent>
                <div className="overflow-x-auto">
                  <div className="min-w-[800px]">
                    <div className="flex border-b border-gray-200 pb-2 mb-4">
                      <div className="w-48 font-medium">Project/Task</div>
                      <div className="flex-1 relative h-8">
                        {Array.from({ length: 12 }).map((_, i) => (
                          <div 
                            key={i} 
                            className="absolute text-xs text-gray-500"
                            style={{ left: `${(i / 12) * 100}%` }}
                          >
                            {new Date(date.getFullYear(), i).toLocaleString('default', { month: 'short' })}
                          </div>
                        ))}
                      </div>
                    </div>
                    {projects.map(project => (
                      <div key={project.id} className="mb-6">
                        <div className="flex items-center h-8">
                          <div className="w-48 font-medium">{project.name}</div>
                          <div className="flex-1 relative h-4 bg-gray-100 rounded">
                            <div 
                              className="absolute h-4 bg-blue-500 rounded"
                              style={{
                                left: `${((project.startDate.getMonth() + (project.startDate.getDate() / 30)) / 12 * 100}%`,
                                width: `${((project.endDate - project.startDate) / (1000 * 60 * 60 * 24 * 30)) / 12 * 100}%`,
                                minWidth: '2px'
                              }}
                            ></div>
                          </div>
                        </div>
                        {getProjectTasks(project.id).map(task => (
                          <div key={task.id} className="flex items-center h-8 ml-4">
                            <div className="w-48 text-sm">{task.name}</div>
                            <div className="flex-1 relative h-4 bg-gray-100 rounded">
                              <div 
                                className={`absolute h-4 rounded ${
                                  task.status === 'completed' ? 'bg-green-500' : 
                                  task.status === 'in-progress' ? 'bg-yellow-500' : 'bg-gray-300'
                                }`}
                                style={{
                                  left: `${((task.startDate.getMonth() + (task.startDate.getDate() / 30)) / 12 * 100}%`,
                                  width: `${((task.endDate - task.startDate) / (1000 * 60 * 60 * 24 * 30)) / 12 * 100}%`,
                                  minWidth: '2px'
                                }}
                              ></div>
                            </div>
                          </div>
                        ))}
                      </div>
                    ))}
                  </div>
                </div>
              </CardContent>
            </Card>
          </TabsContent>

          <TabsContent value="projects" className="mt-6">
            <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
              <Card className="lg:col-span-1">
                <CardHeader>
                  <CardTitle>Add New Project</CardTitle>
                </CardHeader>
                <CardContent>
                  <div className="space-y-4">
                    <Input
                      placeholder="Project name"
                      value={newProject.name}
                      onChange={(e) => setNewProject({...newProject, name: e.target.value})}
                    />
                    <Textarea
                      placeholder="Description"
                      value={newProject.description}
                      onChange={(e) => setNewProject({...newProject, description: e.target.value})}
                    />
                    <Input
                      placeholder="Location"
                      value={newProject.location}
                      onChange={(e) => setNewProject({...newProject, location: e.target.value})}
                    />
                    <div className="grid grid-cols-2 gap-4">
                      <div>
                        <label className="block text-sm font-medium mb-1">Start Date</label>
                        <Calendar
                          mode="single"
                          selected={newProject.startDate}
                          onSelect={(date) => setNewProject({...newProject, startDate: date})}
                          className="rounded-md border"
                        />
                      </div>
                      <div>
                        <label className="block text-sm font-medium mb-1">End Date</label>
                        <Calendar
                          mode="single"
                          selected={newProject.endDate}
                          onSelect={(date) => setNewProject({...newProject, endDate: date})}
                          className="rounded-md border"
                        />
                      </div>
                    </div>
                    <Input
                      type="number"
                      placeholder="Budget ($)"
                      value={newProject.budget}
                      onChange={(e) => setNewProject({...newProject, budget: Number(e.target.value)})}
                    />
                    <Select
                      value={newProject.status}
                      onValueChange={(value) => setNewProject({...newProject, status: value})}
                    >
                      <SelectTrigger>
                        <SelectValue placeholder="Select status" />
                      </SelectTrigger>
                      <SelectContent>
                        <SelectItem value="planning">Planning</SelectItem>
                        <SelectItem value="active">Active</SelectItem>
                        <SelectItem value="on-hold">On Hold</SelectItem>
                        <SelectItem value="completed">Completed</SelectItem>
                      </SelectContent>
                    </Select>
                    <Button onClick={addProject} className="w-full">Create Project</Button>
                  </div>
                </CardContent>
              </Card>

              <div className="lg:col-span-2 space-y-4">
                {projects.map(project => (
                  <Card 
                    key={project.id} 
                    className={`cursor-pointer transition-all hover:shadow-lg ${
                      selectedProject?.id === project.id ? 'ring-2 ring-blue-500' : ''
                    }`}
                    onClick={() => setSelectedProject(project)}
                  >
                    <CardHeader>
                      <div className="flex justify-between">
                        <CardTitle>{project.name}</CardTitle>
                        <span className={`px-2 py-1 rounded-full text-xs ${
                          project.status === 'active' ? 'bg-green-100 text-green-800' :
                          project.status === 'planning' ? 'bg-blue-100 text-blue-800' :
                          project.status === 'on-hold' ? 'bg-yellow-100 text-yellow-800' :
                          'bg-gray-100 text-gray-800'
                        }`}>
                          {project.status.replace('-', ' ')}
                        </span>
                      </div>
                      <CardDescription>{project.description}</CardDescription>
                    </CardHeader>
                    <CardContent>
                      <div className="grid grid-cols-3 gap-4">
                        <div>
                          <div className="text-sm text-gray-500">Location</div>
                          <div>{project.location}</div>
                        </div>
                        <div>
                          <div className="text-sm text-gray-500">Timeline</div>
                          <div>
                            {project.startDate.toLocaleDateString()} - {project.endDate.toLocaleDateString()}
                          </div>
                        </div>
                        <div>
                          <div className="text-sm text-gray-500">Progress</div>
                          <div className="flex items-center gap-2">
                            <Progress 
                              value={calculateProjectProgress(project.id)} 
                              className="h-2 flex-1" 
                            />
                            <span>{calculateProjectProgress(project.id)}%</span>
                          </div>
                        </div>
                      </div>
                    </CardContent>
                  </Card>
                ))}
              </div>
            </div>
          </TabsContent>

          <TabsContent value="tasks" className="mt-6">
            <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
              <Card className="lg:col-span-1">
                <CardHeader>
                  <CardTitle>Add New Task</CardTitle>
                </CardHeader>
                <CardContent>
                  <div className="space-y-4">
                    <Input
                      placeholder="Task name"
                      value={newTask.name}
                      onChange={(e) => setNewTask({...newTask, name: e.target.value})}
                    />
                    <Textarea
                      placeholder="Description"
                      value={newTask.description}
                      onChange={(e) => setNewTask({...newTask, description: e.target.value})}
                    />
                    <Select
                      value={newTask.projectId}
                      onValueChange={(value) => setNewTask({...newTask, projectId: value})}
                    >
                      <SelectTrigger>
                        <SelectValue placeholder="Select project" />
                      </SelectTrigger>
                      <SelectContent>
                        {projects.map(project => (
                          <SelectItem key={project.id} value={project.id}>
                            {project.name}
                          </SelectItem>
                        ))}
                      </SelectContent>
                    </Select>
                    {newTask.projectId && (
                      <Select
                        value={newTask.phase}
                        onValueChange={(value) => setNewTask({...newTask, phase: value})}
                      >
                        <SelectTrigger>
                          <SelectValue placeholder="Select phase" />
                        </SelectTrigger>
                        <SelectContent>
                          {projects.find(p => p.id === newTask.projectId)?.phases.map(phase => (
                            <SelectItem key={phase} value={phase}>{phase}</SelectItem>
                          ))}
                        </SelectContent>
                      </Select>
                    )}
                    <Select
                      value={newTask.assignedTo}
                      onValueChange={(value) => setNewTask({...newTask, assignedTo: value})}
                    >
                      <SelectTrigger>
                        <SelectValue placeholder="Assign to" />
                      </SelectTrigger>
                      <SelectContent>
                        {teamMembers.map(member => (
                          <SelectItem key={member} value={member}>{member}</SelectItem>
                        ))}
                      </SelectContent>
                    </Select>
                    <div className="grid grid-cols-2 gap-4">
                      <div>
                        <label className="block text-sm font-medium mb-1">Start Date</label>
                        <Calendar
                          mode="single"
                          selected={newTask.startDate}
                          onSelect={(date) => setNewTask({...newTask, startDate: date})}
                          className="rounded-md border"
                        />
                      </div>
                      <div>
                        <label className="block text-sm font-medium mb-1">End Date</label>
                        <Calendar
                          mode="single"
                          selected={newTask.endDate}
                          onSelect={(date) => setNewTask({...newTask, endDate: date})}
                          className="rounded-md border"
                        />
                      </div>
                    </div>
                    <Select
                      value={newTask.status}
                      onValueChange={(value) => setNewTask({...newTask, status: value})}
                    >
                      <SelectTrigger>
                        <SelectValue placeholder="Select status" />
                      </SelectTrigger>
                      <SelectContent>
                        <SelectItem value="not-started">Not Started</SelectItem>
                        <SelectItem value="in-progress">In Progress</SelectItem>
                        <SelectItem value="completed">Completed</SelectItem>
                        <SelectItem value="blocked">Blocked</SelectItem>
                      </SelectContent>
                    </Select>
                    <Input
                      type="number"
                      placeholder="Progress %"
                      value={newTask.progress}
                      onChange={(e) => setNewTask({...newTask, progress: Number(e.target.value)})}
                      min="0"
                      max="100"
                    />
                    <Button onClick={addTask} className="w-full">Create Task</Button>
                  </div>
                </CardContent>
              </Card>

              <div className="lg:col-span-2 space-y-4">
                <Card>
                  <CardHeader>
                    <CardTitle>Task List</CardTitle>
                    <CardDescription>
                      {selectedProject ? `Tasks for ${selectedProject.name}` : "All tasks"}
                    </CardDescription>
                  </CardHeader>
                  <CardContent>
                    <DragDropContext onDragEnd={onDragEnd}>
                      <Table>
                        <TableHeader>
                          <TableRow>
                            <TableHead>Task</TableHead>
                            <TableHead>Project</TableHead>
                            <TableHead>Assigned</TableHead>
                            <TableHead>Status</TableHead>
                            <TableHead>Progress</TableHead>
                            <TableHead>Due Date</TableHead>
                            <TableHead>Actions</TableHead>
                          </TableRow>
                        </TableHeader>
                        <Droppable droppableId="tasks">
                          {(provided) => (
                            <TableBody {...provided.droppableProps} ref={provided.innerRef}>
                              {tasks
                                .filter(task => !selectedProject || task.projectId === selectedProject.id)
                                .map((task, index) => (
                                  <Draggable key={task.id} draggableId={task.id.toString()} index={index}>
                                    {(provided) => (
                                      <TableRow 
                                        ref={provided.innerRef}
                                        {...provided.draggableProps}
                                        {...provided.dragHandleProps}
                                      >
                                        <TableCell className="font-medium">{task.name}</TableCell>
                                        <TableCell>{projects.find(p => p.id === task.projectId)?.name}</TableCell>
                                        <TableCell>{task.assignedTo}</TableCell>
                                        <TableCell>
                                          <span className={`px-2 py-1 rounded-full text-xs ${
                                            task.status === 'completed' ? 'bg-green-100 text-green-800' :
                                            task.status === 'in-progress' ? 'bg-yellow-100 text-yellow-800' :
                                            task.status === 'blocked' ? 'bg-red-100 text-red-800' :
                                            'bg-gray-100 text-gray-800'
                                          }`}>
                                            {task.status.replace('-', ' ')}
                                          </span>
                                        </TableCell>
                                        <TableCell>
                                          <div className="flex items-center gap-2">
                                            <Progress value={task.progress} className="h-2 w-24" />
                                            <span>{task.progress}%</span>
                                          </div>
                                        </TableCell>
                                        <TableCell>{task.endDate.toLocaleDateString()}</TableCell>
                                        <TableCell>
                                          <div className="flex gap-2">
                                            <Button 
                                              variant="outline" 
                                              size="sm"
                                              onClick={() => updateTaskStatus(task.id, 'completed')}
                                              disabled={task.status === 'completed'}
                                            >
                                              Complete
                                            </Button>
                                          </div>
                                        </TableCell>
                                      </TableRow>
                                    )}
                                  </Draggable>
                                ))}
                              {provided.placeholder}
                            </TableBody>
                          )}
                        </Droppable>
                      </Table>
                    </DragDropContext>
                  </CardContent>
                </Card>

                {selectedProject && (
                  <Card>
                    <CardHeader>
                      <CardTitle>Critical Path</CardTitle>
                      <CardDescription>
                        Tasks that will delay the project if they're late
                      </CardDescription>
                    </CardHeader>
                    <CardContent>
                      <Table>
                        <TableHeader>
                          <TableRow>
                            <TableHead>Task</TableHead>
                            <TableHead>Depends On</TableHead>
                            <TableHead>Start Date</TableHead>
                            <TableHead>End Date</TableHead>
                          </TableRow>
                        </TableHeader>
                        <TableBody>
                          {getCriticalPath(selectedProject.id).map(task => (
                            <TableRow key={task.id}>
                              <TableCell className="font-medium">{task.name}</TableCell>
                              <TableCell>
                                {task.dependencies.length > 0 ? 
                                  tasks.find(t => t.id === task.dependencies[0])?.name : 'None'}
                              </TableCell>
                              <TableCell>{task.startDate.toLocaleDateString()}</TableCell>
                              <TableCell>{task.endDate.toLocaleDateString()}</TableCell>
                            </TableRow>
                          ))}
                        </TableBody>
                      </Table>
                    </CardContent>
                  </Card>
                )}
              </div>
            </div>
          </TabsContent>

          <TabsContent value="documents" className="mt-6">
            <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
              <Card className="lg:col-span-1">
                <CardHeader>
                  <CardTitle>Upload Documents</CardTitle>
                </CardHeader>
                <CardContent>
                  <div className="space-y-4">
                    <Select
                      value={selectedProject?.id || ''}
                      onValueChange={(value) => setSelectedProject(projects.find(p => p.id === value))}
                    >
                      <SelectTrigger>
                        <SelectValue placeholder="Select project" />
                      </SelectTrigger>
                      <SelectContent>
                        {projects.map(project => (
                          <SelectItem key={project.id} value={project.id}>
                            {project.name}
                          </SelectItem>
                        ))}
                      </SelectContent>
                    </Select>
                    <div className="border-2 border-dashed rounded-lg p-6 text-center">
                      <label className="flex flex-col items-center gap-2 cursor-pointer">
                        <Upload className="h-8 w-8 text-gray-500" />
                        <span className="font-medium">Drag and drop files here</span>
                        <span className="text-sm text-gray-500">or click to browse</span>
                        <input 
                          type="file" 
                          className="hidden" 
                          multiple
                          onChange={handleFileUpload}
                        />
                      </label>
                    </div>
                  </div>
                </CardContent>
              </Card>

              <div className="lg:col-span-2">
                <Card>
                  <CardHeader>
                    <CardTitle>Project Documents</CardTitle>
                    <CardDescription>
                      {selectedProject ? `Documents for ${selectedProject.name}` : "Select a project to view documents"}
                    </CardDescription>
                  </CardHeader>
                  <CardContent>
                    {selectedProject ? (
                      <Table>
                        <TableHeader>
                          <TableRow>
                            <TableHead>Name</TableHead>
                            <TableHead>Type</TableHead>
                            <TableHead>Size</TableHead>
                            <TableHead>Uploaded</TableHead>
                          </TableRow>
                        </TableHeader>
                        <TableBody>
                          {documents
                            .filter(doc => doc.projectId === selectedProject.id)
                            .map(doc => (
                              <TableRow key={doc.id}>
                                <TableCell className="font-medium">{doc.name}</TableCell>
                                <TableCell>{doc.type}</TableCell>
                                <TableCell>{Math.round(doc.size / 1024)} KB</TableCell>
                                <TableCell>{doc.uploaded.toLocaleDateString()}</TableCell>
                              </TableRow>
                            ))}
                        </TableBody>
                      </Table>
                    ) : (
                      <div className="text-center py-8 text-gray-500">
                        Please select a project to view its documents
                      </div>
                    )}
                  </CardContent>
                </Card>
              </div>
            </div>
          </TabsContent>

          <TabsContent value="team" className="mt-6">
            <Card>
              <CardHeader>
                <CardTitle>Team Management</CardTitle>
                <CardDescription>Manage your construction team members</CardDescription>
              </CardHeader>
              <CardContent>
                <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
                  <Card>
                    <CardHeader>
                      <CardTitle>Add Team Member</CardTitle>
                    </CardHeader>
                    <CardContent>
                      <div className="space-y-4">
                        <Input placeholder="Full name" />
                        <Input placeholder="Email" />
                        <Input placeholder="Role" />
                        <Select>
                          <SelectTrigger>
                            <SelectValue placeholder="Select specialty" />
                          </SelectTrigger>
                          <SelectContent>
                            <SelectItem value="carpentry">Carpentry</SelectItem>
                            <SelectItem value="electrical">Electrical</SelectItem>
                            <SelectItem value="plumbing">Plumbing</SelectItem>
                            <SelectItem value="masonry">Masonry</SelectItem>
                            <SelectItem value="management">Project Management</SelectItem>
                          </SelectContent>
                        </Select>
                        <Button className="w-full">Add Member</Button>
                      </div>
                    </CardContent>
                  </Card>

                  <div className="md:col-span-2">
                    <Card>
                      <CardHeader>
                        <CardTitle>Team Members</CardTitle>
                      </CardHeader>
                      <CardContent>
                        <Table>
                          <TableHeader>
                            <TableRow>
                              <TableHead>Name</TableHead>
                              <TableHead>Assigned Projects</TableHead>
                              <TableHead>Tasks</TableHead>
                              <TableHead>Actions</TableHead>
                            </TableRow>
                          </TableHeader>
                          <TableBody>
                            {teamMembers.map(member => (
                              <TableRow key={member}>
                                <TableCell className="font-medium">{member}</TableCell>
                                <TableCell>
                                  {projects.filter(p => p.team.includes(member)).length}
                                </TableCell>
                                <TableCell>
                                  {tasks.filter(t => t.assignedTo === member).length}
                                </TableCell>
                                <TableCell>
                                  <Button variant="outline" size="sm">View</Button>
                                </TableCell>
                              </TableRow>
                            ))}
                          </TableBody>
                        </Table>
                      </CardContent>
                    </Card>
                  </div>
                </div>
              </CardContent>
            </Card>
          </TabsContent>
        </Tabs>
      </div>
    </div>
  );
}
