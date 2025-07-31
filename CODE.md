import React, { createContext, useContext, useState, useEffect } from 'react';
import { User, GraduationCap, Users, Plus, Edit2, Trash2, LogOut, Search, Filter } from 'lucide-react';

// ===== CONTEXTS =====
const AuthContext = createContext();

// ===== SERVICES =====
// Mock Supabase service for demo
const supabaseService = {
  auth: {
    signUp: async (email, password) => {
      // Mock implementation
      return { user: { id: '1', email }, error: null };
    },
    signIn: async (email, password) => {
      if (email === 'demo@example.com' && password === 'demo123') {
        return { user: { id: '1', email: 'demo@example.com' }, error: null };
      }
      return { user: null, error: { message: 'Invalid credentials' } };
    },
    signOut: async () => {
      return { error: null };
    },
    getUser: () => {
      const stored = localStorage.getItem('auth_user');
      return stored ? JSON.parse(stored) : null;
    }
  },
  
  classes: {
    getAll: async () => {
      const stored = localStorage.getItem('classes');
      return stored ? JSON.parse(stored) : [];
    },
    create: async (classData) => {
      const classes = await supabaseService.classes.getAll();
      const newClass = { ...classData, id: Date.now().toString() };
      const updated = [...classes, newClass];
      localStorage.setItem('classes', JSON.stringify(updated));
      return newClass;
    },
    update: async (id, classData) => {
      const classes = await supabaseService.classes.getAll();
      const updated = classes.map(c => c.id === id ? { ...c, ...classData } : c);
      localStorage.setItem('classes', JSON.stringify(updated));
      return updated.find(c => c.id === id);
    },
    delete: async (id) => {
      const classes = await supabaseService.classes.getAll();
      const updated = classes.filter(c => c.id !== id);
      localStorage.setItem('classes', JSON.stringify(updated));
      return true;
    }
  },
  
  students: {
    getAll: async () => {
      const stored = localStorage.getItem('students');
      return stored ? JSON.parse(stored) : [];
    },
    create: async (studentData) => {
      const students = await supabaseService.students.getAll();
      const newStudent = { ...studentData, id: Date.now().toString() };
      const updated = [...students, newStudent];
      localStorage.setItem('students', JSON.stringify(updated));
      return newStudent;
    },
    update: async (id, studentData) => {
      const students = await supabaseService.students.getAll();
      const updated = students.map(s => s.id === id ? { ...s, ...studentData } : s);
      localStorage.setItem('students', JSON.stringify(updated));
      return updated.find(s => s.id === id);
    },
    delete: async (id) => {
      const students = await supabaseService.students.getAll();
      const updated = students.filter(s => s.id !== id);
      localStorage.setItem('students', JSON.stringify(updated));
      return true;
    }
  }
};

// ===== HOOKS =====
const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};

const useClasses = () => {
  const [classes, setClasses] = useState([]);
  const [loading, setLoading] = useState(false);

  const fetchClasses = async () => {
    setLoading(true);
    try {
      const data = await supabaseService.classes.getAll();
      setClasses(data);
    } catch (error) {
      console.error('Error fetching classes:', error);
    } finally {
      setLoading(false);
    }
  };

  const createClass = async (classData) => {
    try {
      const newClass = await supabaseService.classes.create(classData);
      setClasses(prev => [...prev, newClass]);
      return newClass;
    } catch (error) {
      console.error('Error creating class:', error);
      throw error;
    }
  };

  const updateClass = async (id, classData) => {
    try {
      const updatedClass = await supabaseService.classes.update(id, classData);
      setClasses(prev => prev.map(c => c.id === id ? updatedClass : c));
      return updatedClass;
    } catch (error) {
      console.error('Error updating class:', error);
      throw error;
    }
  };

  const deleteClass = async (id) => {
    try {
      await supabaseService.classes.delete(id);
      setClasses(prev => prev.filter(c => c.id !== id));
    } catch (error) {
      console.error('Error deleting class:', error);
      throw error;
    }
  };

  useEffect(() => {
    fetchClasses();
  }, []);

  return { classes, loading, createClass, updateClass, deleteClass, fetchClasses };
};

const useStudents = () => {
  const [students, setStudents] = useState([]);
  const [loading, setLoading] = useState(false);

  const fetchStudents = async () => {
    setLoading(true);
    try {
      const data = await supabaseService.students.getAll();
      setStudents(data);
    } catch (error) {
      console.error('Error fetching students:', error);
    } finally {
      setLoading(false);
    }
  };

  const createStudent = async (studentData) => {
    try {
      const newStudent = await supabaseService.students.create(studentData);
      setStudents(prev => [...prev, newStudent]);
      return newStudent;
    } catch (error) {
      console.error('Error creating student:', error);
      throw error;
    }
  };

  const updateStudent = async (id, studentData) => {
    try {
      const updatedStudent = await supabaseService.students.update(id, studentData);
      setStudents(prev => prev.map(s => s.id === id ? updatedStudent : s));
      return updatedStudent;
    } catch (error) {
      console.error('Error updating student:', error);
      throw error;
    }
  };

  const deleteStudent = async (id) => {
    try {
      await supabaseService.students.delete(id);
      setStudents(prev => prev.filter(s => s.id !== id));
    } catch (error) {
      console.error('Error deleting student:', error);
      throw error;
    }
  };

  useEffect(() => {
    fetchStudents();
  }, []);

  return { students, loading, createStudent, updateStudent, deleteStudent, fetchStudents };
};

// ===== UI COMPONENTS =====
const Button = ({ children, variant = 'primary', size = 'md', onClick, disabled, className = '', ...props }) => {
  const baseClasses = 'font-medium rounded-lg transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed';
  
  const variants = {
    primary: 'bg-blue-600 hover:bg-blue-700 text-white focus:ring-blue-500',
    secondary: 'bg-gray-200 hover:bg-gray-300 text-gray-800 focus:ring-gray-500',
    danger: 'bg-red-600 hover:bg-red-700 text-white focus:ring-red-500',
    ghost: 'hover:bg-gray-100 text-gray-600 focus:ring-gray-500'
  };
  
  const sizes = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-sm',
    lg: 'px-6 py-3 text-base'
  };

  return (
    <button
      className={`${baseClasses} ${variants[variant]} ${sizes[size]} ${className}`}
      onClick={onClick}
      disabled={disabled}
      {...props}
    >
      {children}
    </button>
  );
};

const Modal = ({ isOpen, onClose, title, children }) => {
  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
      <div className="bg-white rounded-lg shadow-xl max-w-md w-full max-h-[90vh] overflow-y-auto">
        <div className="px-6 py-4 border-b border-gray-200">
          <h3 className="text-lg font-semibold text-gray-900">{title}</h3>
        </div>
        <div className="p-6">
          {children}
        </div>
      </div>
    </div>
  );
};

const LoadingSpinner = () => (
  <div className="flex justify-center items-center p-8">
    <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600"></div>
  </div>
);

// ===== AUTH COMPONENTS =====
const LoginForm = () => {
  const { signIn, signUp } = useAuth();
  const [isLogin, setIsLogin] = useState(true);
  const [email, setEmail] = useState('demo@example.com');
  const [password, setPassword] = useState('demo123');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    setError('');

    try {
      if (isLogin) {
        await signIn(email, password);
      } else {
        await signUp(email, password);
      }
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="max-w-md w-full space-y-8 p-8">
        <div className="text-center">
          <GraduationCap className="mx-auto h-12 w-12 text-blue-600" />
          <h2 className="mt-6 text-3xl font-bold text-gray-900">
            {isLogin ? 'Sign in to your account' : 'Create new account'}
          </h2>
        </div>
        <form className="mt-8 space-y-6" onSubmit={handleSubmit}>
          {error && (
            <div className="bg-red-50 border border-red-200 text-red-600 px-4 py-3 rounded">
              {error}
            </div>
          )}
          <div>
            <label htmlFor="email" className="block text-sm font-medium text-gray-700">
              Email address
            </label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              required
              className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500"
            />
          </div>
          <div>
            <label htmlFor="password" className="block text-sm font-medium text-gray-700">
              Password
            </label>
            <input
              id="password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              required
              className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500"
            />
          </div>
          <Button
            type="submit"
            className="w-full"
            disabled={loading}
          >
            {loading ? 'Processing...' : (isLogin ? 'Sign In' : 'Sign Up')}
          </Button>
          <div className="text-center">
            <button
              type="button"
              onClick={() => setIsLogin(!isLogin)}
              className="text-blue-600 hover:text-blue-500 text-sm"
            >
              {isLogin ? "Don't have an account? Sign up" : 'Already have an account? Sign in'}
            </button>
          </div>
        </form>
        <div className="mt-4 p-4 bg-blue-50 rounded-lg">
          <p className="text-sm text-blue-700 font-medium">Demo Credentials:</p>
          <p className="text-sm text-blue-600">Email: demo@example.com</p>
          <p className="text-sm text-blue-600">Password: demo123</p>
        </div>
      </div>
    </div>
  );
};

// ===== CLASS COMPONENTS =====
const ClassForm = ({ classData, onSubmit, onCancel }) => {
  const [formData, setFormData] = useState({
    name: classData?.name || '',
    academic_year: classData?.academic_year || '',
    is_active: classData?.is_active ?? true
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Class Name
        </label>
        <input
          type="text"
          value={formData.name}
          onChange={(e) => setFormData({ ...formData, name: e.target.value })}
          required
          className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="e.g., Grade 5A"
        />
      </div>
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Academic Year
        </label>
        <input
          type="text"
          value={formData.academic_year}
          onChange={(e) => setFormData({ ...formData, academic_year: e.target.value })}
          required
          className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="e.g., 2024-2025"
        />
      </div>
      <div className="flex items-center">
        <input
          type="checkbox"
          id="is_active"
          checked={formData.is_active}
          onChange={(e) => setFormData({ ...formData, is_active: e.target.checked })}
          className="h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded"
        />
        <label htmlFor="is_active" className="ml-2 text-sm text-gray-700">
          Active Class
        </label>
      </div>
      <div className="flex space-x-3 pt-4">
        <Button type="submit" className="flex-1">
          {classData ? 'Update Class' : 'Create Class'}
        </Button>
        <Button type="button" variant="secondary" onClick={onCancel} className="flex-1">
          Cancel
        </Button>
      </div>
    </form>
  );
};

const ClassCard = ({ classItem, onEdit, onDelete }) => (
  <div className="bg-white p-6 rounded-lg shadow-sm border border-gray-200 hover:shadow-md transition-shadow">
    <div className="flex justify-between items-start mb-4">
      <div>
        <h3 className="text-lg font-semibold text-gray-900">{classItem.name}</h3>
        <p className="text-sm text-gray-600">{classItem.academic_year}</p>
      </div>
      <div className="flex space-x-2">
        <Button variant="ghost" size="sm" onClick={() => onEdit(classItem)}>
          <Edit2 className="h-4 w-4" />
        </Button>
        <Button variant="ghost" size="sm" onClick={() => onDelete(classItem.id)}>
          <Trash2 className="h-4 w-4 text-red-500" />
        </Button>
      </div>
    </div>
    <div className="flex items-center justify-between">
      <span className={`inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium ${
        classItem.is_active 
          ? 'bg-green-100 text-green-800' 
          : 'bg-gray-100 text-gray-800'
      }`}>
        {classItem.is_active ? 'Active' : 'Inactive'}
      </span>
    </div>
  </div>
);

const ClassList = () => {
  const { classes, loading, createClass, updateClass, deleteClass } = useClasses();
  const [showForm, setShowForm] = useState(false);
  const [editingClass, setEditingClass] = useState(null);

  const handleSubmit = async (formData) => {
    try {
      if (editingClass) {
        await updateClass(editingClass.id, formData);
      } else {
        await createClass(formData);
      }
      setShowForm(false);
      setEditingClass(null);
    } catch (error) {
      console.error('Error saving class:', error);
    }
  };

  const handleEdit = (classItem) => {
    setEditingClass(classItem);
    setShowForm(true);
  };

  const handleDelete = async (id) => {
    if (window.confirm('Are you sure you want to delete this class?')) {
      await deleteClass(id);
    }
  };

  if (loading) return <LoadingSpinner />;

  return (
    <div>
      <div className="flex justify-between items-center mb-6">
        <h2 className="text-2xl font-bold text-gray-900">Classes</h2>
        <Button onClick={() => setShowForm(true)}>
          <Plus className="h-4 w-4 mr-2" />
          Add Class
        </Button>
      </div>

      {classes.length === 0 ? (
        <div className="text-center py-12">
          <GraduationCap className="mx-auto h-12 w-12 text-gray-400" />
          <h3 className="mt-2 text-sm font-medium text-gray-900">No classes</h3>
          <p className="mt-1 text-sm text-gray-500">Get started by creating a new class.</p>
        </div>
      ) : (
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          {classes.map((classItem) => (
            <ClassCard
              key={classItem.id}
              classItem={classItem}
              onEdit={handleEdit}
              onDelete={handleDelete}
            />
          ))}
        </div>
      )}

      <Modal
        isOpen={showForm}
        onClose={() => {
          setShowForm(false);
          setEditingClass(null);
        }}
        title={editingClass ? 'Edit Class' : 'Create New Class'}
      >
        <ClassForm
          classData={editingClass}
          onSubmit={handleSubmit}
          onCancel={() => {
            setShowForm(false);
            setEditingClass(null);
          }}
        />
      </Modal>
    </div>
  );
};

// ===== STUDENT COMPONENTS =====
const StudentForm = ({ studentData, onSubmit, onCancel }) => {
  const { classes } = useClasses();
  const [formData, setFormData] = useState({
    name: studentData?.name || '',
    email: studentData?.email || '',
    class_id: studentData?.class_id || '',
    date_of_birth: studentData?.date_of_birth || '',
    phone: studentData?.phone || ''
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Student Name
        </label>
        <input
          type="text"
          value={formData.name}
          onChange={(e) => setFormData({ ...formData, name: e.target.value })}
          required
          className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="Enter student name"
        />
      </div>
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Email
        </label>
        <input
          type="email"
          value={formData.email}
          onChange={(e) => setFormData({ ...formData, email: e.target.value })}
          className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="student@example.com"
        />
      </div>
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Class
        </label>
        <select
          value={formData.class_id}
          onChange={(e) => setFormData({ ...formData, class_id: e.target.value })}
          required
          className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          <option value="">Select a class</option>
          {classes.map((classItem) => (
            <option key={classItem.id} value={classItem.id}>
              {classItem.name} - {classItem.academic_year}
            </option>
          ))}
        </select>
      </div>
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Date of Birth
        </label>
        <input
          type="date"
          value={formData.date_of_birth}
          onChange={(e) => setFormData({ ...formData, date_of_birth: e.target.value })}
          className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
      </div>
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Phone
        </label>
        <input
          type="tel"
          value={formData.phone}
          onChange={(e) => setFormData({ ...formData, phone: e.target.value })}
          className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="Phone number"
        />
      </div>
      <div className="flex space-x-3 pt-4">
        <Button type="submit" className="flex-1">
          {studentData ? 'Update Student' : 'Add Student'}
        </Button>
        <Button type="button" variant="secondary" onClick={onCancel} className="flex-1">
          Cancel
        </Button>
      </div>
    </form>
  );
};

const StudentTable = ({ students, classes, onEdit, onDelete }) => {
  const getClassName = (classId) => {
    const classItem = classes.find(c => c.id === classId);
    return classItem ? classItem.name : 'Unknown';
  };

  return (
    <div className="overflow-x-auto">
      <table className="min-w-full divide-y divide-gray-200">
        <thead className="bg-gray-50">
          <tr>
            <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
              Name
            </th>
            <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
              Email
            </th>
            <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
              Class
            </th>
            <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
              Phone
            </th>
            <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
              Actions
            </th>
          </tr>
        </thead>
        <tbody className="bg-white divide-y divide-gray-200">
          {students.map((student) => (
            <tr key={student.id} className="hover:bg-gray-50">
              <td className="px-6 py-4 whitespace-nowrap">
                <div className="text-sm font-medium text-gray-900">{student.name}</div>
              </td>
              <td className="px-6 py-4 whitespace-nowrap">
                <div className="text-sm text-gray-500">{student.email}</div>
              </td>
              <td className="px-6 py-4 whitespace-nowrap">
                <div className="text-sm text-gray-900">{getClassName(student.class_id)}</div>
              </td>
              <td className="px-6 py-4 whitespace-nowrap">
                <div className="text-sm text-gray-500">{student.phone}</div>
              </td>
              <td className="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
                <div className="flex space-x-2">
                  <Button variant="ghost" size="sm" onClick={() => onEdit(student)}>
                    <Edit2 className="h-4 w-4" />
                  </Button>
                  <Button variant="ghost" size="sm" onClick={() => onDelete(student.id)}>
                    <Trash2 className="h-4 w-4 text-red-500" />
                  </Button>
                </div>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};

const StudentList = () => {
  const { students, loading, createStudent, updateStudent, deleteStudent } = useStudents();
  const { classes } = useClasses();
  const [showForm, setShowForm] = useState(false);
  const [editingStudent, setEditingStudent] = useState(null);
  const [searchTerm, setSearchTerm] = useState('');
  const [selectedClass, setSelectedClass] = useState('');

  const filteredStudents = students.filter(student => {
    const matchesSearch = student.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
                         student.email.toLowerCase().includes(searchTerm.toLowerCase());
    const matchesClass = selectedClass === '' || student.class_id === selectedClass;
    return matchesSearch && matchesClass;
  });

  const handleSubmit = async (formData) => {
    try {
      if (editingStudent) {
        await updateStudent(editingStudent.id, formData);
      } else {
        await createStudent(formData);
      }
      setShowForm(false);
      setEditingStudent(null);
    } catch (error) {
      console.error('Error saving student:', error);
    }
  };

  const handleEdit = (student) => {
    setEditingStudent(student);
    setShowForm(true);
  };

  const handleDelete = async (id) => {
    if (window.confirm('Are you sure you want to delete this student?')) {
      await deleteStudent(id);
    }
  };

  if (loading) return <LoadingSpinner />;

  return (
    <div>
      <div className="flex justify-between items-center mb-6">
        <h2 className="text-2xl font-bold text-gray-900">Students</h2>
        <Button onClick={() => setShowForm(true)}>
          <Plus className="h-4 w-4 mr-2" />
          Add Student
        </Button>
      </div>

      <div className="mb-6 flex flex-col sm:flex-row gap-4">
        <div className="flex-1">
          <div className="relative">
            <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400 h-4 w-4" />
            <input
              type="text"
              placeholder="Search students..."
              value={searchTerm}
              onChange={(e) => setSearchTerm(e.target.value)}
              className="pl-10 pr-4 py-2 w-full border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>
        </div>
        <div className="sm:w-48">
          <select
            value={selectedClass}
            onChange={(e) => setSelectedClass(e.target.value)}
            className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option value="">All Classes</option>
            {classes.map((classItem) => (
              <option key={classItem.id} value={classItem.id}>
                {classItem.name}
              </option>
            ))}
          </select>
        </div>
      </div>

      {filteredStudents.length === 0 ? (
        <div className="text-center py-12">
          <Users className="mx-auto h-12 w-12 text-gray-400" />
          <h3 className="mt-2 text-sm font-medium text-gray-900">No students found</h3>
          <p className="mt-1 text-sm text-gray-500">
            {students.length === 0 
              ? 'Get started by adding a new student.' 
              : 'Try adjusting your search or filter criteria.'
            }
          </p>
        </div>
      ) : (
        <div className="bg-white shadow rounded-lg">
          <StudentTable
            students={filteredStudents}
            classes={classes}
            onEdit={handleEdit}
            onDelete={handleDelete}
          />
        </div>
      )}

      <Modal
        isOpen={showForm}
        onClose={() => {
          setShowForm(false);
          setEditingStudent(null);
        }}
        title={editingStudent ? 'Edit Student' : 'Add New Student'}
      >
        <StudentForm
          studentData={editingStudent}
          onSubmit={handleSubmit}
          onCancel={() => {
            setShowForm(false);
            setEditingStudent(null);
          }}
        />
      </Modal>
    </div>
  );
};

// ===== LAYOUT COMPONENTS =====
const Header = () => {
  const { user, signOut } = useAuth();

  return (
    <header className="bg-white shadow-sm border-b border-gray-200">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex justify-between items-center h-16">
          <div className="flex items-center">
            <GraduationCap className="h-8 w-8 text-blue-600" />
            <h1 className="ml-2 text-xl font-bold text-gray-900">Student Management</h1>
          </div>
          <div className="flex items-center space-x-4">
            <span className="text-sm text-gray-700">Welcome, {user?.email}</span>
            <Button variant="ghost" size="sm" onClick={signOut}>
              <LogOut className="h-4 w-4 mr-2" />
              Sign Out
            </Button>
          </div>
        </div>
      </div>
    </header>
  );
};

const Navigation = ({ activeTab, setActiveTab }) => {
  const tabs = [
    { id: 'classes', label: 'Classes', icon: GraduationCap },
    { id: 'students', label: 'Students', icon: Users }
  ];

  return (
    <nav className="bg-white shadow-sm">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex space-x-8">
          {tabs.map((tab) => {
            const Icon = tab.icon;
            return (
              <button
                key={tab.id}
                onClick={() => setActiveTab(tab.id)}
                className={`flex items-center px-3 py-4 text-sm font-medium border-b-2 transition-colors ${
                  activeTab === tab.id
                    ? 'border-blue-500 text-blue-600'
                    : 'border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300'
                }`}
              >
                <Icon className="h-4 w-4 mr-2" />
                {tab.label}
              </button>
            );
          })}
        </div>
      </div>
    </nav>
  );
};

const Layout = ({ children }) => (
  <div className="min-h-screen bg-gray-50">
    <Header />
    {children}
  </div>
);

// ===== MAIN APP COMPONENT =====
const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Check for existing session
    const storedUser = supabaseService.auth.getUser();
    if (storedUser) {
      setUser(storedUser);
    }
    setLoading(false);
  }, []);

  const signIn = async (email, password) => {
    const { user, error } = await supabaseService.auth.signIn(email, password);
    if (error) throw new Error(error.message);
    
    setUser(user);
    localStorage.setItem('auth_user', JSON.stringify(user));
  };

  const signUp = async (email, password) => {
    const { user, error } = await supabaseService.auth.signUp(email, password);
    if (error) throw new Error(error.message);
    
    setUser(user);
    localStorage.setItem('auth_user', JSON.stringify(user));
  };

  const signOut = async () => {
    await supabaseService.auth.signOut();
    setUser(null);
    localStorage.removeItem('auth_user');
    localStorage.removeItem('classes');
    localStorage.removeItem('students');
  };

  const value = {
    user,
    signIn,
    signUp,
    signOut,
    loading
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};

const Dashboard = () => {
  const [activeTab, setActiveTab] = useState('classes');

  return (
    <Layout>
      <Navigation activeTab={activeTab} setActiveTab={setActiveTab} />
      <main className="max-w-7xl mx-auto py-6 px-4 sm:px-6 lg:px-8">
        {activeTab === 'classes' && <ClassList />}
        {activeTab === 'students' && <StudentList />}
      </main>
    </Layout>
  );
};

const App = () => {
  return (
    <AuthProvider>
      <AppContent />
    </AuthProvider>
  );
};

const AppContent = () => {
  const { user, loading } = useAuth();

  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <LoadingSpinner />
      </div>
    );
  }

  return user ? <Dashboard /> : <LoginForm />;
};

export default App;